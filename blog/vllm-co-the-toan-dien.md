---
title: 'vLLM từ trong ra ngoài: Kiến trúc, tham số cấu hình, tuning và best practices'
description: >-
  Bài viết chi tiết về cơ chế hoạt động của vLLM, ý nghĩa từng tham số cấu hình,
  cách tune cho throughput/latency, custom model và best practices khi deploy
  production trên GPU H100.
pubDate: '2026-06-23'
lang: vi
tags:
  - vllm
  - llm
  - inference
  - gpu
  - quantization
concepts:
  - PagedAttention
  - Continuous Batching
  - KV Cache
  - Prefix Caching
  - Speculative Decoding
  - Chunked Prefill
  - Tensor Parallelism
  - Quantization
tools:
  - vLLM
  - NVIDIA CUDA
  - Docker
  - HuggingFace
topics:
  - LLM & Inference
readTime: 25
---
Bạn vừa tải xong một model 26B từ HuggingFace. Chạy thử bằng `transformers`, sinh văn bản ngon lành. Nhưng khi mở API ra cho 50 người dùng cùng lúc, server chết đứng — mỗi request mất 30 giây, GPU thì idle 90% thời gian. Bạn nhìn vào `nvidia-smi`, VRAM còn hơn nửa trống, GPU util gần 0%. Có gì đó rất sai.

Đó là lúc vLLM xuất hiện. Cùng con GPU đó, cùng model đó, vLLM đẩy throughput lên gấp 2-4 lần so với các serving system cũ như FasterTransformer hay TGI. Nhưng để đạt được con số đó, bạn không thể cứ để mặc định rồi hy vọng — cần hiểu vLLM làm gì bên trong, và từng tham số cấu hình ảnh hưởng đến hiệu năng ra sao.

Bài này đi từ kiến trúc bên trong vLLM đến cách tune tham số cho từng loại workload, custom model khi vLLM chưa hỗ trợ native, và những bài học xương máu khi deploy production. Mình sẽ dùng một server thật làm ví dụ xuyên suốt: H100 NVL 96GB đang chạy Gemma-4-26B-A4B-NVFP4 trong Docker, config lấy từ `docker inspect` chứ không bịa.

## VLLM làm gì bên trong?

Hiểu vLLM bằng cách nhìn vào bốn kỹ thuật cốt lõi. Mọi tham số cấu hình đều xoay quanh việc điều khiển bốn thứ này.

### PagedAttention — hoặc: tại sao VRAM lại bị lãng phí

Khi LLM sinh văn bản, nó cần lưu **KV cache** — bộ nhớ tạm chứa kết quả attention của từng token đã xử lý. Nếu không có KV cache, mỗi lần sinh token mới, model phải đọc lại toàn bộ prompt từ đầu. Với KV cache, nó chỉ cần nhìn lại "bộ nhớ tạm" đó.

Vấn đề: KV cache to cực nhanh. Một request 4K token trên model 70B ngốn 2-4 GB VRAM chỉ cho cache. Nếu bạn cấp phát VRAM theo cách ngây thơ nhất — dành sẵn một dải liên tục đủ lớn cho worst case — thì phần lớn thời gian VRAM bị bỏ trống. Giống như đặt trước 10 ghế cho mỗi khách trong nhà hàng, khách chỉ ngồi 1 ghế, 9 ghế kia trống nhưng không ai được ngồi.

PagedAttention giải quyết bằng cách chia VRAM thành các block nhỏ — mỗi block chứa KV cache của 16 token. Khi request cần thêm, engine xin thêm block. Khi request xong, block trả về pool. Cách này giống hệt cách hệ điều hành quản lý RAM: không dành sẵn cả đống cho mỗi process, mà cấp phát theo trang (page) khi cần.

![PagedAttention: Block-based VRAM so với Contiguous](/images/diagram-paged-attention.png)

Kết quả: cùng lượng VRAM, vLLM phục vụ được 2-4 lần nhiều request song song hơn so với cấp phát liên tục truyền thống. Đây là lý do vLLM ra đời — [bài paper gốc](https://arxiv.org/abs/2309.06180) tại SOSP 2023 của nhóm UC Berkeley đã chứng minh con số này.

*Cách vLLM quản lý KV cache theo block — ảnh từ bài "Inside vLLM" chính thức:*

![KV Cache Blocks — PagedAttention (vLLM official)](/images/vllm-pagedattention-blocks.png)

### Continuous batching — đừng chờ ai cả

Batching truyền thống (static batching) gom một nhóm request, xử lý chung, rồi **đợi tất cả xong** mới nhận nhóm tiếp. Nếu request A sinh 10 token, request B sinh 500 token, cả nhóm chờ B. GPU thì idle, user thì bực.

Continuous batching khác ở một điểm đơn giản: mỗi lần model sinh thêm một token (mỗi "iteration"), engine kiểm tra xem request nào đã xong. Xong thì trả kết quả ngay, giải phóng chỗ cho request mới từ hàng đợi. Không ai phải chờ ai.

Hình dung thế này: thay vì xả cả bồn tắm rồi bơm lại từng lần (static batching), bồn tắm luôn đầy nước. Người vào người ra liên tục, GPU không bao giờ nhàn rỗi khi còn hàng đợi.

![Continuous Batching vs Static Batching](/images/diagram-continuous-batching.png)

Tham số `--max-num-seqs` quyết định "kích thước bồn tắm" — bao nhiêu request được bơi cùng lúc. Set quá nhỏ, GPU idle. Set quá lớn, mỗi request chia quá ít VRAM, dễ OOM.

### Prefix caching — đừng tính lại cái đã tính

Hầu hết chatbot bắt đầu mọi conversation bằng cùng một system prompt. Mỗi API call gửi system prompt y hệt nhau, nhưng engine vẫn tính KV cache từ đầu cho mỗi lần. Lãng phí.

Prefix caching lưu lại KV cache của đoạn đầu chung. Request mới có cùng prefix? Tái sử dụng ngay, bỏ qua prefill cho đoạn đã cache. Trên server thật, prefix cache hit rate thường đạt 40-60% — gần một nửa số token không cần tính lại.

Từ V1 engine (phiên bản mặc định từ cuối 2024), prefix caching bật sẵn, không cần flag.

*Prefix caching tái sử dụng KV cache của system prompt chung — ảnh từ vLLM:*

![Prefix Caching (vLLM official)](/images/vllm-prefix-cache.png)

### Chunked prefill — giết hai chim một đá

Inference có hai giai đoạn rất khác nhau:

**Prefill** — xử lý toàn bộ prompt đầu vào. Tốn nhiều compute (tính toán) vì phải chạy attention trên mọi token cùng lúc. Nhưng chạy song song được, nên nhanh.

**Decode** — sinh token mới từng cái một. Ít compute nhưng nhiều memory bandwidth (đọc KV cache). Không song song được vì token sau cần token trước.

Trước đây, engine tách hai giai đoạn: hoặc đang prefill (GPU compute tải nặng) hoặc đang decode (GPU compute idle, chỉ đọc memory). Lãng phí ở cả hai phía.

Chunked prefill trộn cả hai trong cùng một batch: vài request đang prefill (ăn compute), vài request đang decode (ăn bandwidth). GPU tận dụng đồng thời cả hai loại tài nguyên. Throughput tăng thêm 20-40% tùy workload — con số này từ [vLLM v0.3 release announcement](https://blog.vllm.ai/2024/01/23/vllm-v0-3-0.html).

*Chunked prefill trộn prefill và decode trong cùng batch — ảnh từ vLLM:*

![Chunked Prefill (vLLM official)](/images/vllm-chunked-prefill.png)

## Bốn tham số quyết định 90% hiệu năng

vLLM có hơn 100 tham số. Nhưng nếu bạn chỉ hiểu bốn cái dưới đây, bạn đã config tốt hơn 80% người dùng.

### `--gpu-memory-utilization` — cái van VRAM

Tham số này cho vLLM biết: "mày được dùng bao nhiêu phần trăm VRAM tổng của GPU". Mặc định 0.9 (90%).

Engine tính: `VRAM cho KV cache = (Tổng VRAM × utilization) − Model weights − Overhead`. Phần KV cache này quyết định bao nhiêu request song song được phục vụ.

**0.9** an toàn cho hầu hết trường hợp — để lại 10% cho CUDA context, activation tensors tạm, v.v.

**0.95** khi server chuyên chạy một mình, không chia GPU với process nào. Ép thêm được chút VRAM cho KV cache.

**0.85 trở xuống** khi chạy chung GPU với process khác (monitoring, embedding service).

Nhưng có một cạm bẫy Docker: nếu bạn chạy hai container cùng map GPU 0, vLLM trong container A **không biết** container B đang xài VRAM. Nó chỉ thấy tổng VRAM 96GB và lấy 90% = 86GB. Container B cũng lấy 86GB. Tổng cần 172GB trên GPU 96GB → OOM. Luôn set `--gpu-memory-utilization` thấp hơn khi chia GPU.

Trên server H100 thật: `--gpu-memory-utilization 0.9` cho VRAM pool khoảng 86GB. Model Gemma-4-26B-NVFP4 dùng khoảng 18GB (NVFP4 quantized). Còn lại ~68GB cho KV cache — đủ phục vụ 64 request song song với context 100K token mỗi request.

### `--max-num-seqs` — kích thước bồn tắm

Số request tối đa xử lý song song. Mặc định 256.

**Quá thấp (vd 8)**: GPU idle, throughput thấp. Server xử lý ít request dù VRAM còn nhiều trống.

**Quá cao (vd 1024 trên GPU nhỏ)**: mỗi request được chia quá ít VRAM, KV cache đầy, engine dành nhiều thời gian swap (chuyển cache ra CPU RAM) hơn tính toán. Gọi là thrashing — giống RAM máy tính đầy, ổ cứng quay không ngừng nhưng máy chậm như rùa.

Cách chọn: đo workload. Chatbot (câu ngắn, cần phản hồi nhanh) — 32-128. Batch processing (xử lý hàng loạt, không cần real-time) — 128-512 trở lên.

Trên server thật: `--max-num-seqs 64`. Đây là chatbot nên 64 request song song là đủ. KV cache usage log hiển thị 0-1.4% — nghĩa là server còn **rất nhiều** dư địa, có thể tăng `max-num-seqs` lên 128-256 nếu cần.

### `--max-model-len` — đừng để cao hơn cần

Tổng token tối đa cho một request (input + output). Mặc định lấy từ model config — có thể là 32K, 128K, hoặc hơn.

Nhiều người để mặc định rồi không hiểu tại sao server chỉ phục vụ được vài request song song. Lý do: vLLM **dành sẵn** KV cache cho worst case. Nếu `max-model-len` = 32768, mỗi request được giành 32K token KV cache — dù thực tế user chỉ chat 500 token. VRAM bị "đặt trước" vô ích.

**Quy tắc**: đo độ dài request thực tế (p95 hoặc p99), rồi set `max-model-len` = p99 + buffer 20%. Chatbot hiếm khi vượt 8K. Batch processing thường 4K. Đừng set cao hơn cần.

Trên server thật: `--max-model-len 100000`. Đây là model multimodal (Gemma-4 xử lý cả ảnh), cần context dài cho visual tokens. Với 100K context, mỗi request **có thể** chiếm rất nhiều KV cache, nhưng vì `max-num-seqs` chỉ 64 nên tổng vẫn nằm trong VRAM budget.

### Prefix caching (V1: mặc định on)

Từ V1 engine, prefix caching bật sẵn. Không cần config gì. Nhưng nếu hit rate thấp (dưới 20%), có thể do prompt không có prefix chung, hoặc prompt bị thay đổi nhỏ ở đầu (số dòng few-shot khác nhau, dấu cách thừa, v.v.). Prefix cache match **chính xác** từng token từ đầu — một token khác là miss toàn bộ.

## Các tham số quan trọng còn lại

### Batching & Scheduler

`--max-num-batched-tokens` (mặc định 8192) — tổng token tối đa trong một batch. Đây không giống `max-num-seqs`: một batch có 64 request đang decode thì chứa 64 token (1 mỗi request). Nhưng nếu 1 request đang prefill 4096 token, batch đó chứa 4096 + 63 = 4159 token. Nếu `max-num-batched-tokens` = 4096, request prefill bị chia nhỏ thành chunks (đó chính là chunked prefill).

`--scheduler-policy` — `lpm` (Longest Prefix Match, mặc định) ưu tiên request có prefix match nhiều nhất, hoặc `fcfs` (First Come First Serve) xử lý theo thứ tự.

### Quantization — nén model để chạy trên GPU nhỏ hơn

Quantization giảm độ chính xác weights: FP16 (16-bit) → INT8 (8-bit) → INT4 (4-bit). Model nhỏ hơn, inference nhanh hơn, VRAM ít hơn. Đổi lại: chất lượng giảm nhẹ.

**AWQ** ([paper](https://arxiv.org/abs/2306.00978)) và **GPTQ** ([paper](https://arxiv.org/abs/2210.17323)) (4-bit) phổ biến nhất, giữ 95-99% chất lượng. Model 70B từ 140GB FP16 xuống 35-40GB, chạy vừa trên một GPU 80GB.

**FP8** (8-bit float) — native trên H100 và Ada Lovelace. Gần như không mất chất lượng, tốc độ tốt nhất. Đây là hướng đi chính cho GPU thế hệ mới.

**NVFP4** (4-bit float) — format mới nhất của NVIDIA. Server thật đang dùng: `Gemma-4-26B-A4B-NVFP4` nén model 26B từ ~52GB FP16 xuống ~18GB, chạy thoải mái trên H100 với hơn 68GB VRAM còn lại cho KV cache.

**BitsAndBytes** (NF4/INT8) — dễ dùng nhất, chuyển đổi từ HuggingFace chỉ cần vài dòng code, nhưng chậm hơn AWQ/GPTQ.

### Đa GPU — khi một GPU không đủ

Model 70B FP16 cần 140GB VRAM — không GPU đơn nào có đủ. vLLM hỗ trợ ba cách chia:

**Tensor Parallelism** (`--tensor-parallel-size N`): chia mỗi layer thành N phần, mỗi GPU giữ một phần. Yêu cầu NVLink (GPU giao tiếp nhanh). Tăng nhẹ latency (giao tiếp giữa GPU) nhưng chia VRAM. Phổ biến nhất cho inference.

**Pipeline Parallelism** (`--pipeline-parallel-size N`): chia theo layer — GPU 1 xử lý layer 1-32, GPU 2 xử lý layer 33-64. Ít dùng cho inference vì gây "bong bóng" — GPU 2 chờ GPU 1 xong mới bắt đầu.

**Data Parallelism** (`--data-parallel-size N`): chạy N bản sao model trên N GPU, chia request đều. Đơn giản nhất, không cần NVLink. Khi cần scale lên nhiều server, đây là cách dễ nhất.

### Tool calling và reasoning

`--enable-auto-tool-choice` bật tool calling (function calling) — model có thể gọi external function.

`--tool-call-parser` chọn parser: `hermes`, `gemma4`, `llama3_json`, `mistral`, v.v. — mỗi model có format tool call khác nhau.

`--reasoning-parser` tách thinking tokens (reasoning) ra riêng: `deepseek_r1`, `gemma4`, v.v. Cần thiết cho model có "chain of thought" (chuỗi suy luận) tích hợp.

Trên server thật, cả ba đều bật:
```
--reasoning-parser gemma4
--enable-auto-tool-choice
--tool-call-parser gemma4
```
Gemma-4 có reasoning tích hợp (sinh thinking tokens) và hỗ trợ tool calling. Parser `gemma4` biết cách tách thinking ra riêng và parse tool call đúng format.

### Speculative decoding — dùng model nhỏ đoán trước

Ý tưởng: dùng model nhỏ (draft model) đoán trước 5-10 token, rồi model lớn xác nhận. Nếu đoán đúng, sinh được nhiều token cùng lúc. Nếu sai, bỏ qua và tính lại.

Hiệu quả khi: model lớn và draft model nhỏ (tỉ lệ size ≥ 5x), workload generation-heavy (nhiều output), và hai model cùng vocabulary.

Config: `--speculative-model "tinyllama-1b"` + `--num-speculative-tokens 5`.

Không phải lúc nào cũng nhanh hơn — nếu acceptance rate (tỉ lệ đoán đúng) thấp, overhead của draft model lớn hơn gain. Đo trước khi dùng.

*Speculative decoding: draft model đoán trước, target model xác nhận — ảnh từ vLLM:*

![Speculative Decoding (vLLM official)](/images/vllm-speculative-decoding.png)

## Tune theo workload — không có config tối ưu cho mọi trường hợp

### Chatbot real-time (cần TTFT thấp)

Time-to-first-token (TTFT) — thời gian từ lúc user gửi đến lúc nhận token đầu. Với chatbot, cảm nhận "nhanh" hay "chậm" phụ thuộc chủ yếu vào TTFT.

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --max-num-seqs 64 \
  --max-model-len 8192 \
  --max-num-batched-tokens 4096
```

`max-num-seqs 64` thay vì 256 — chatbot không cần batch lớn, mỗi request cần KV cache dồi dào để không chờ. `max-num-batched-tokens 4096` — batch nhỏ hơn giúp prefill nhanh hơn, giảm TTFT.

### Batch processing (cần throughput tối đa)

Latency không quan trọng. Throughput là tất cả.

```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct \
  --tensor-parallel-size 2 \
  --gpu-memory-utilization 0.95 \
  --max-num-seqs 512 \
  --max-model-len 4096 \
  --max-num-batched-tokens 16384
```

`gpu-memory-utilization 0.95` — ép hết VRAM vì chạy batch-only. `max-num-seqs 512` — batch lớn. `max-model-len 4096` — batch processing thường xử lý văn bản ngắn.

### Multimodal (vision-language)

Model xử lý ảnh (như Gemma-4) cần config thêm. Hình ảnh được convert thành visual tokens, ngốn KV cache nhiều hơn text rất nhiều.

Trên server thật, config Docker Compose:
```yaml
command:
  [
    "--model", "/gemma4-nvfp4/Gemma-4-26B-A4B-NVFP4",
    "--served-model-name", "gemma-4-26B-A4B-NVFP4",
    "--gpu-memory-utilization", "0.9",
    "--max-num-seqs", "64",
    "--reasoning-parser", "gemma4",
    "--enable-auto-tool-choice",
    "--tool-call-parser", "gemma4",
    "--mm-processor-kwargs", "{\"max_soft_tokens\": 560}",
    "--max-model-len", "100000"
  ]
```

`--mm-processor-kwargs '{"max_soft_tokens": 560}'` — giới hạn số visual token mỗi ảnh ở 560, tránh OOM khi user gửi ảnh độ phân giải cao. `max-num-seqs 64` thay vì 256 vì mỗi request multimodal ngốn nhiều VRAM hơn.

## Custom model khi vLLM chưa hỗ trợ

### Fine-tune từ kiến trúc đã có — dễ nhất

Fine-tune từ Llama, Qwen, Mistral, Gemma và chỉ thay weights? vLLM đọc `config.json`, nhận diện kiến trúc, dùng implementation có sẵn:

```bash
vllm serve /path/to/my-finetuned-llama --trust-remote-code
```

`--trust-remote-code` cần khi model có file Python custom (vd `modeling_my_model.py`).

### Kiến trúc mới hoặc sửa đổi — cần code

Nếu kiến trúc khác bản gốc (thêm layer, đổi attention), cần implement trong vLLM. Hai cách:

**Plugin (không sửa vLLM)**: đăng ký class kế thừa từ kiến trúc đã có, override phần thay đổi:

```python
from vllm.model_executor.models import register_model
from vllm.model_executor.models.llama import LlamaForCausalLM

@register_model("MyCustomModel")
class MyCustomModel(LlamaForCausalLM):
    def forward(self, *args, **kwargs):
        # thêm modifications ở đây
        return super().forward(*args, **kwargs)
```

Phù hợp khi model là biến thể nhẹ của kiến trúc đã hỗ trợ.

**Implement từ đầu**: với kiến trúc hoàn toàn mới — tạo file trong `vllm/model_executor/models/`, kế thừa base classes (`PagedAttention`, v.v.), đăng ký trong `registry.py`, build từ source (`pip install -e .`). vLLM docs có hướng dẫn chi tiết ở mục "Adding a New Model".

### Quantization format tùy chỉnh

Nếu model dùng quantization vLLM chưa support, đăng ký custom:

```python
from vllm.quantization import register_quantization_config

@register_quantization_config("my_quant")
class MyQuantConfig:
    def get_quant_method(self, layer, prefix):
        ...
```

Rồi chạy `--quantization my_quant`.

### Chat template tùy chỉnh

Model custom cần template riêng để format input:

```bash
vllm serve /path/to/model --chat-template /path/to/template.jinja
```

## Deploy production — những thứ không ai nói cho bạn

Sơ đồ tổng quan kiến trúc production trên H100:

![vLLM Production Stack](/images/diagram-production-stack.png)

### Docker: `ipc: host` là bắt buộc

vLLM dùng shared memory (shm) cho inter-process communication. Docker mặc định giới hạn shm 64MB ([Docker docs](https://docs.docker.com/compose/compose-file/05-services-reference/#shm_size)) — quá nhỏ. Không set `ipc: host`, vLLM sẽ OOM hoặc crash mà không có thông báo rõ ràng.

```yaml
services:
  vllm:
    image: vllm/vllm-openai:v0.22.1
    ipc: host              # ← KHÔNG ĐƯỢC QUÊN
    restart: always
    ports:
      - "30000:8000"
    volumes:
      - /path/to/model:/model:ro
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["0"]
              capabilities: [gpu]
```

Đây chính là config đang chạy trên H100 thật — port 30000 map ra 8000 trong container, model mount read-only, GPU 0 dedicated.

### Health check: cho thời gian load model

Model 70B mất 2-5 phút load weights vào VRAM. Health check mặc định của Docker (bắt đầu check ngay) sẽ báo unhealthy khi server đang load:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 300s    # cho 5 phút load model
```

### Graceful shutdown

vLLM xử lý SIGTERM — đợi request đang chạy xong rồi mới thoát. Nhưng Docker kill sau 10s mặc định. Set `stop_grace_period: 300s` để không ngắt giữa chừng.

### Version pinning — đừng dùng `:latest`

vLLM phát triển nhanh, breaking changes xảy ra giữa các minor version. Luôn pin version cụ thể: `vllm/vllm-openai:v0.22.1` (như server thật đang dùng). Test kỹ trước khi upgrade.

### Không bao giờ expose API không có auth

Endpoint `/v1/completions` cho phép ai cũng gọi được model. Không có API key = ai cũng xài GPU miễn phí của bạn. Luôn đặt sau reverse proxy với TLS + rate limiting, hoặc tối thiểu:

```bash
--api-keys "sk-you...here"
```

### Monitoring — đọc engine logs

VLLM in ra metrics mỗi 10 giây. Trên server thật, log trông thế này:

```
Engine 000: Avg prompt throughput: 2743.7 tokens/s,
Avg generation throughput: 151.9 tokens/s,
Running: 0 reqs, Waiting: 0 reqs,
GPU KV cache usage: 0.0%,
Prefix cache hit rate: 49.8%,
MM cache hit rate: 23.3%
```

Đọc thế nào:

- **KV cache usage 0%**: server đang idle, không có request. Nếu **luôn** 100%, server quá tải — giảm `max-model-len` hoặc tăng VRAM.
- **Prefix cache hit rate 50%**: một nửa token prompt được tái sử dụng. Tốt. Nếu dưới 20%, prompt không có prefix chung — cân nhắc tắt prefix caching (trong V0) hoặc chuẩn hóa prompt format.
- **Running vs Waiting**: nếu Waiting luôn > 0, server quá tải. Cần scale lên (thêm GPU hoặc thêm instance).
- **Generation throughput**: nếu thấp bất thường (dưới 50 tokens/s trên H100), kiểm tra GPU util — có thể batch quá nhỏ hoặc model bị bottleneck ở đâu đó.

### Khi nào KHÔNG nên dùng vLLM?

- **Model dưới 1B**: overhead của PagedAttention và scheduler lớn hơn lợi ích. Dùng `transformers` hoặc Ollama.
- **Embedding-only**: vLLM tập trung vào generation. Dùng Text Embeddings Inference (TEI) hoặc Infinity.
- **Edge/mobile**: vLLM thiết kế cho datacenter GPU. Trên laptop, dùng llama.cpp hoặc Ollama.
- **Batch size luôn = 1**: nếu chỉ một user dùng, PagedAttention và continuous batching không có ý nghĩa. Dùng Ollama.
- **Cần determinism tuyệt đối**: vLLM có thể cho kết quả hơi khác nhau giữa các run do batching nondeterministic. Với nghiên cứu cần reproducibility 100%, dùng `transformers` với seed cố định.

## Kết

vLLM không phải "magic box" — nó là bốn kỹ thuật (PagedAttention, continuous batching, prefix caching, chunked prefill) được điều phối bởi một scheduler thông minh. Hiểu bốn kỹ thuật này, bạn config đúng tham số thay vì copy-paste từ đâu đó và hy vọng.

Nguyên tắc tuning: **đo trước, tune sau**. Mở engine logs, đọc KV cache usage, throughput, latency thực tế — rồi điều chỉnh tham số dựa trên số liệu. Mỗi workload có "sweet spot" khác nhau.

Khi deploy production: Docker + `ipc: host` + health check + version pinning. Đơn giản nhưng đủ cho 90% use cases. Khi cần scale, chạy nhiều instance (data parallelism) là cách dễ nhất trước khi thử tensor parallelism.
