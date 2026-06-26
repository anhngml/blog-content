---
title: 'Opik: Mở đèn trong phòng tối — LLM Observability từ A đến Z'
description: >-
  Phân tích sâu Opik — nền tảng open-source LLM observability từ Comet. Trace,
  span, LLM-as-a-Judge, CI/CD evaluation, production monitoring. So sánh
  LangSmith, Langfuse, Arize Phoenix. Khi nào cần và khi nào chưa cần.
pubDate: '2026-06-26'
lang: vi
tags:
  - opik
  - llm
  - observability
  - mlops
  - evaluation
concepts:
  - LLM Observability
  - Distributed Tracing
  - LLM-as-a-Judge
  - Hallucination Detection
  - RAG Evaluation
  - Online Evaluation
  - Evaluation-Driven Development
  - OpenTelemetry
tools:
  - Opik
  - ClickHouse
  - Docker
  - OpenTelemetry
topics:
  - MLOps
  - Monitoring & Observability
readTime: 12
---
Bạn xây một RAG chatbot. Ba tháng trước, nó trả lời rất tốt — chính xác, nhanh, rẻ. Hôm nay, người dùng phàn nàn nó "bịa nhiều hơn". Bạn đổi prompt, thử model mới, cập nhật vector DB. Chatbot "có vẻ" tốt hơn... nhưng bạn không chắc. Không có con số, không có baseline, không có cách nào biết thay đổi giúp hay hại.

Đây là vấn đề cốt lõi khi vận hành LLM trong production: **bạn không nhìn thấy gì**. Không có stack trace rõ ràng như code thông thường. Không có log request/response có cấu trúc. Một câu trả lời sai có thể do retrieval kém, prompt xấu, model hallucinate, hay đơn giản context window bị thiếu. Không có quan sát (observability), bạn đi mò trong bóng tối.

[Opik](https://github.com/comet-ml/opik) — open-source từ Comet, giấy phép Apache 2.0 — giải quyết đúng khoảng trống này. Nó không phải dashboard log thuần tuý, mà là một nền tảng **quan sát, đánh giá, và tối ưu** LLM application end-to-end: từ khi prototype ở laptop đến khi chạy 40 triệu traces/ngày trong production.

Bài này đi sâu vào cách Opik hoạt động, các khái niệm cốt lõi (trace, span, LLM-as-a-Judge), và tại sao nó đáng cân nhắc khi bạn cần "ánh sáng" cho LLM pipeline của mình.

## Bài toán: LLM khác gì so với phần mềm truyền thống?

Trong một web service truyền thống, có request đến, code xử lý, database trả về, response đi. Bạn instrument bằng OpenTelemetry — mỗi request sinh ra một *trace* (chuỗi thao tác), mỗi thao tác là một *span* (đơn vị đo). Xem trace, bạn biết bottleneck nằm đâu.

LLM application phức tạp hơn nhiều. Một request "Thủ đô Việt Nam là gì?" đi qua:

1. **Retrieval**: query embedding → vector search → top-k documents
2. **Prompt construction**: ghép context + question thành prompt
3. **LLM call**: gửi prompt tới GPT-4o, nhận tokens
4. **Post-processing**: moderation check, format response
5. **Potential hallucination**: model bịa ra câu trả lời không có trong context

![Trace and Span — xương sống của LLM observability](/images/diagram-trace-span.png)

Mỗi bước có thể thất bại theo cách riêng: retrieval trả tài liệu sai, prompt quá dài gây tràn context, model bịa, moderation chặn nhầm. Và bạn cần biết không chỉ *rằng* nó sai, mà *tại sao*, *ở bước nào*, *với tần suất bao nhiêu*.

Đó là lý do [LLM observability](https://www.braintrust.dev/articles/llm-observability-guide) ra đời — không chỉ trace latency, mà trace **chất lượng**: câu trả lời có bị ảo giác không? Có trả lời đúng câu hỏi không? Context có chính xác không?

## Kiến trúc Opik: SDK → Server → Dashboard

Opik có kiến trúc ba lớp rõ ràng, và điểm hay là **bạn có thể self-host toàn bộ**.

![Kiến trúc Opik — từ SDK đến Dashboard](/images/diagram-architecture.png)

**Lớp 1: SDK** — Bạn thêm decorator `@opik.track` vào hàm Python, hoặc dùng integration có sẵn (60+ framework: OpenAI, LangChain, LlamaIndex, Anthropic, Gemini, CrewAI, Ollama...). SDK tự động capture: input, output, số token, latency, cost, model name. Nếu hàm gọi hàm con, trace tự tạo tree — đúng như distributed tracing truyền thống.

```python
import opik

opik.configure(use_local=True)  # self-hosted local

@opik.track
def rag_chatbot(question: str) -> str:
    docs = retrieve(question)      # ← tự động trở span con
    answer = llm_call(question, docs)  # ← thêm span con nữa
    return answer
```

Chỉ cần thêm decorator, mỗi lần gọi hàm sinh ra một trace đầy đủ. Không cần sửa logic code.

**Lớp 2: Backend Server** — Viết bằng Python (FastAPI), lưu Traces vào [ClickHouse](https://clickhouse.com/) — column-store database tối ưu cho analytics workload với tốc độ ghi cao (Opik thiết kế cho 40M+ traces/ngày). Metadata (datasets, projects, users) lưu MySQL. Backend cũng chạy **LLM-as-a-Judge engine** — chấm điểm chất lượng từng trace bằng một LLM giám khảo.

**Lớp 3: Dashboard (React)** — Trace Explorer (xem tree spans, drill-down), Evaluation Results (score per experiment), Production Dashboard (token usage, feedback scores theo thời gian), Prompt Playground (thử prompt, so sánh model), Datasets & Experiments (quản lý test cases).

Ba cách triển khai: Comet Cloud (đăng ký dùng ngay, free tier), Docker Compose (`./opik.sh` cho local dev), hoặc Kubernetes + Helm (production self-hosted). Điểm đáng chú ý: **phiên bản open-source có đầy đủ tính năng**, không phải "lite version" như nhiều tool khác.

## Trace và Span: Mở đèn trong phòng tối

Nếu bạn đã quen với [OpenTelemetry](https://opentelemetry.io/docs/concepts/observability-primer/), trace và span trong Opik rất giống — nhưng mở rộng cho đặc thù LLM.

Một **trace** đại diện cho một request hoàn chỉnh — ví dụ, một lần user hỏi chatbot. Trace chứa metadata: project name, timestamp, tổng token, tổng cost, tổng latency.

Một **span** đại diện cho một thao tác bên trong trace. Span `retrieve_documents()` ghi lại: query input, 3 documents output, thời gian 0.45 giây. Span `openai.chat.completions.create()` ghi: model `gpt-4o`, input 380 tokens, output 70 tokens, cost $0.003, latency 1.2 giây.

Trace là cây (tree), span là nút (node). Trace root chứa toàn bộ, span con lồng nhau theo thứ tự gọi. Khi bạn click vào trace trong Dashboard, thấy ngay: retrieval chậm? LLM tốn nhiều token? Moderation fail?

Điểm hay: Opik hỗ trợ **[OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/)** — chuẩn mới cho tracing LLM, định nghĩa attribute tên gì (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`). Điều này nghĩa là trace từ Opik tương thích với các OTel collector khác — không bị lock-in.

## LLM-as-a-Judge: Dùng LLM để chấm điểm LLM

Đây là tính năng quan trọng nhất của Opik và là điểm khác biệt cốt lõi so với monitoring truyền thống.

Trong software thường, "đúng/sai" thường nhị phân — test pass hoặc fail. Với LLM, "đúng" liên tục và mơ hồ: câu trả lời có thể đúng facts nhưng sai tone, đúng tone nhưng bịa, hoặc đúng nhưng không trả lời đúng câu hỏi. Bạn cần đánh giá đa chiều.

[LLM-as-a-Judge](https://www.confident-ai.com/blog/llm-evaluation-metrics-everything-you-need-for-llm-evaluation) — dùng một LLM mạnh (thường GPT-4o hoặc Claude) để chấm điểm output của LLM application — giải quyết vấn đề này. Thay vì đánh giá thủ công hàng ngàn câu trả lời, Judge LLM chạy tự động với tiêu chí rõ ràng.

![LLM-as-a-Judge — dùng LLM để chấm điểm LLM](/images/diagram-llm-as-judge.png)

Opik đi kèm sẵn các metric:

**Hallucination** — Judge LLM nhận output và context, trả lời: "Câu trả lời có thông tin không có trong context không?" Score 0 = hoàn toàn có nền tảng, 1 = hoàn toàn bịa. Đây là metric quan trọng nhất cho RAG.

**Answer Relevance** — "Câu trả lời có thực sự giải quyết câu hỏi không?" Score 0 = lạc đề, 1 = trả lời đúng trọng tâm.

**Context Precision** — "Các tài liệu retrieved có được xếp hạng đúng không? Tài liệu liên quan có xuất hiện ở top không?" Đánh giá chất lượng retrieval, không phải generation.

**Moderation** — "Câu trả lời có nội dung độc hại, nhạy cảm không?" Score 0 = an toàn, 1 = vi phạm.

**Faithfulness** — Tương tự hallucination nhưng chi tiết hơn: tách output thành từng claim, kiểm tra từng claim có nền tảng từ context không.

```python
from opik.evaluation.metrics import Hallucination

metric = Hallucination()
score = metric.score(
    input="Thủ đô Pháp là gì?",
    output="Paris",
    context=["Paris là thủ đô của Pháp..."]
)
print(score)  # → 0.05 (rất thấp = tốt)
```

Ngoài ra, bạn có thể tạo **custom metric** — viết Python heuristic hoặc prompt Judge LLM theo tiêu chí riêng (vd: "câu trả lời có dùng giọng điệu thân thiện không?"). Opik cũng hỗ trợ **heuristic metrics** không cần LLM: exact match, BLEU/ROUGE, regex check, token count.

## CI/CD Evaluation: Chặn regression trước production

Giống như bạn chạy unit test trước khi merge PR, Opik cho phép chạy **evaluation test** trước khi deploy thay đổi prompt/model. Đây là bước chuyển từ "hy vọng nó tốt hơn" sang "chứng minh bằng số liệu".

![Tích hợp Opik vào CI/CD pipeline](/images/diagram-cicd-comparison.png)

Quy trình [evaluation-driven development](https://langfuse.com/blog/2025-10-21-testing-llm-applications):

1. Tạo **Dataset** — tập test cases (câu hỏi + câu trả lời chuẩn, hoặc chỉ câu hỏi).
2. Chạy app qua từng test case → thu thập output.
3. Chấm điểm output bằng LLM-as-a-Judge.
4. So sánh với **baseline** — version trước có score trung bình bao nhiêu? Version mới có bị regression không?
5. Gate: nếu score tụt dưới threshold → **chặn merge**.

Opik tích hợp trực tiếp với [pytest](https://www.comet.com/docs/opik/testing/pytest_integration/) — viết eval test như viết unit test:

```python
def test_rag_hallucination():
    dataset = opik.get_dataset("rag_test_set")
    eval_result = evaluate(
        experiment_name="rag-v2",
        dataset=dataset,
        task=rag_chatbot,
        scoring_metrics=[Hallucination(), AnswerRelevance()],
    )
    assert eval_result.avg_scores["hallucination"] < 0.2
```

Mỗi lần chạy sinh ra một **Experiment** — lưu kết quả trong Opik để so sánh version. Bạn trả lời được: "Đổi từ GPT-4o sang Claude có giảm hallucination không?" bằng data, không cảm tính.

So với các tool khác trong cùng không gian, Opik có vị trí riêng:

| Tool | License | Điểm mạnh chính |
|---|---|---|
| **Opik** | Apache 2.0 | Self-host đầy đủ, Agent Optimizer, 60+ integrations |
| **LangSmith** | Commercial | Tích hợp sâu LangChain/LangGraph, annotation queue |
| **Langfuse** | MIT | Self-host friendly, eval ecosystem mạnh |
| **Arize Phoenix** | Apache 2.0 | OTel-native, notebook-integrated UI |
| **Braintrust** | Commercial | Eval-first, CI/CD gates, 1M spans free |

Opik nổi bật ở **Agent Optimizer** — SDK riêng để tối ưu prompt và tool cho agent, và **Guardrails** — tính năng runtime safety. Cùng với 60+ integration native (Google ADK, Autogen, CrewAI...), độ bao phủ framework rộng nhất trong các tool open-source.

## Production Monitoring: Khi LLM chạy 24/7

Trace và eval là cho dev phase. Khi LLM vào production, bạn cần monitoring liên tục — và đây là lúc 40M traces/ngày ClickHouse thực sự tỏa sáng.

![Production Monitoring với Opik](/images/diagram-production-monitoring.png)

Production Dashboard hiển thị real-time: trace count, token cost, P95 latency, feedback scores theo thời gian. Nhưng số liệu thô chưa đủ — Opik thêm **Online Evaluation Rules**.

Online Rule chạy LLM-as-a-Judge trên **mẫu** traces production (không phải tất cả — quá đắt) theo định kỳ. Khi phát hiện pattern bất thường, nó alert:

- "Hallucination score trung bình > 0.5 trong 1 giờ qua" → alert đỏ
- "P95 latency > 2 giây kể từ deploy 14:30" → alert vàng
- "Moderation pass rate giảm xuống 95%" → kiểm tra ngay

Điều này chuyển monitoring LLM từ "phản ứng" (user phàn nàn → mới biết lỗi) sang "chủ động" (phát hiện regression trước khi user thấy). Đặc biệt quan trọng khi model provider thay đổi phiên bản ngầm — GPT-4o bản tháng 6 có thể khác bản tháng 3 — mà bạn không hay biết cho đến khi chất lượng tụt.

## ClickHouse: Vì sao 40M traces/ngày khả thi?

Chi tiết kỹ thuật đáng chú ý: Opik chọn [ClickHouse](https://clickhouse.com/) làm storage layer cho traces, không phải PostgreSQL hay Elasticsearch. ClickHouse là **column-store OLAP database** — tối ưu cho workload ghi nhiều, đọc analytics (GROUP BY, aggregate trên hàng triệu dòng).

Trace data có đặc điểm: ghi liên tục (mỗi LLM call = 1+ trace), nhưng đọc theo nhóm (dashboard aggregate, không SELECT * từng dòng). ClickHouse phù hợp pattern này hơn row-store (PostgreSQL) nhiều bậc — đây là lý do Opik đạt 40M+ traces/ngày trên hạ tầng khiêm tốn.

Từ version 1.7.0, Opik chuyển sang **ClickHouse replicated tables** — cho phép horizontal scaling và high availability khi self-hosted trên production.

## Khi nào nên dùng Opik?

Không phải ai cũng cần LLM observability ngay. Nếu bạn mới viết prototype, gọi OpenAI API vài chục lần — log ra console là đủ. Nhưng khi:

- **LLM application vào production** với user thật — bạn cần biết regression ngay, không đợi phàn nàn.
- **RAG pipeline phức tạp** (retrieval + reranking + generation + guardrails) — cần trace từng bước để debug.
- **Đổi model/prompt thường xuyên** — cần baseline để so sánh version, không đoán mò.
- **Agent system nhiều bước** (LangGraph, CrewAI) — trace là cách duy nhất hiểu agent "nghĩ gì".
- **Tuân thủ compliance** — cần audit trail: ai hỏi gì, AI trả lời gì, khi nào.

Opik đặc biệt hấp dẫn nếu:
- Bạn cần **self-host** (dữ liệu nhạy cảm, air-gap) — Apache 2.0, Docker/K8s đầy đủ.
- Bạn dùng **nhiều framework khác nhau** — 60+ integration là rộng nhất hiện nay.
- Bạn muốn **CI/CD evaluation** — pytest integration chặt chẽ.

Nhưng cũng cần lưu ý hạn chế:
- **Resource requirement** — ClickHouse + MySQL + Backend + Frontend = 4 container minimum. Không "nhẹ".
- **Self-hosted = tự vận hành** — upgrade, backup, monitoring hạ tầng là trách nhiệm bạn. Comet Cloud giải tỏa việc này nếu OK dùng cloud.
- **LLM-as-a-Judge tốn token** — mỗi eval call tốn thêm LLM call. Với dataset lớn, chi phí cộng dồn. Sample thay vì eval toàn bộ nếu cần.

## Kết: Nhìn thấy, rồi mới tối ưu được

Vận hành LLM trong production mà không có observability giống lái xe bịt mắt. Bạn không biết khi nào đâm, và khi đâm rồi cũng không biết tại sao. Opik — và các tool cùng loại như Langfuse, Arize Phoenix, LangSmith — cung cấp ánh sáng.

Trace cho bạn thấy **bước nào chậm, bước nào sai**. LLM-as-a-Judge cho bạn biết **chất lượng bao nhiêu, tốt hay xấu đi**. CI/CD evaluation chặn **regression trước khi tới user**. Production monitoring cảnh báo **khi thứ gì đó thay đổi** mà bạn không hay biết.

Điểm khác biệt của Opik: open-source thật (Apache 2.0, không "open-core" giới hạn), self-host đầy đủ tính năng, 60+ integration framework, và Agent Optimizer + Guardrails — hai tính năng runtime mà nhiều tool thiếu. Đổi lại, bạn cần hạ tầng (ClickHouse không nhẹ) và công sức vận hành nếu self-host.

Trong thế giới LLM nơi mọi thứ thay đổi từng tháng — model mới, prompt mới, framework mới — observability không phải "nice to have". Nó là cần thiết để trả lời câu hỏi duy nhất quan trọng: **phiên bản này có thực sự tốt hơn phiên trước không?** Opik cho bạn công cụ để trả lời bằng số liệu, không cảm tính.

---

**Tham khảo:**
- Repo: [comet-ml/opik](https://github.com/comet-ml/opik) (Apache 2.0)
- [Opik Documentation](https://www.comet.com/docs/opik/)
- [LLM Observability Guide (Braintrust)](https://www.braintrust.dev/articles/llm-observability-guide)
- [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/)
- [LLM Evaluation Metrics (Confident AI)](https://www.confident-ai.com/blog/llm-evaluation-metrics-everything-you-need-for-llm-evaluation)
- [Testing LLM Applications (Langfuse)](https://langfuse.com/blog/2025-10-21-testing-llm-applications)
- [Best LLM Observability Tools 2026 (Firecrawl)](https://www.firecrawl.dev/blog/best-llm-observability-tools)
- [ClickHouse](https://clickhouse.com/)
