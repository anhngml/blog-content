---
title: RAG là gì? Giải thích Retrieval-Augmented Generation từ đầu
description: >-
  Tìm hiểu RAG từ đầu: cách hoạt động, tại sao cần thiết, pipeline chi tiết và
  các tool phổ biến nhất hiện nay
pubDate: 2026-05-10T00:00:00.000Z
tags:
  - RAG
  - LLM
  - Vector DB
  - AI
lang: vi
concepts:
  - RAG
  - Vector DB
  - Embedding
  - Semantic Search
  - Chunking
tools:
  - LangChain
  - LlamaIndex
  - Qdrant
  - ChromaDB
  - OpenAI
topics:
  - AI
  - RAG
relations:
  - to: vector-databases
    type: PREREQUISITE
  - to: llm-agents
    type: SAME_TOPIC
---

## RAG là gì?

RAG (Retrieval-Augmented Generation) là kỹ thuật kết hợp **khả năng tìm kiếm thông tin** (retrieval) với **khả năng sinh văn bản** (generation) của LLM.

Vấn đề cốt lõi mà RAG giải quyết: LLM có kiến thức bị giới hạn ở thời điểm training. Chúng không biết về tài liệu nội bộ của bạn, không cập nhật được thông tin mới nhất.

### Vấn đề trước RAG

```
User: "Doanh thu Q4 của công ty là bao nhiêu?"
LLM:  "Tôi không có thông tin về doanh thu công ty bạn."
```

### Với RAG

```
User query
  → Embed query → Search vector DB → Tìm tài liệu liên quan
  → Đưa tài liệu vào context LLM
  → LLM trả lời dựa trên tài liệu thực tế
```

## Pipeline RAG chi tiết

### Bước 1: Indexing (làm một lần)

1. **Load** tài liệu (PDF, Word, HTML, Markdown...)
2. **Chunk** — chia nhỏ thành các đoạn 512-1024 tokens
3. **Embed** — chuyển mỗi chunk thành vector số học
4. **Store** — lưu vectors vào vector database

### Bước 2: Retrieval (mỗi query)

1. **Embed query** — chuyển câu hỏi thành vector
2. **Similarity search** — tìm k chunks gần nhất trong DB
3. **Rerank** (optional) — sắp xếp lại kết quả

### Bước 3: Generation

```python
prompt = f"""
Dựa vào các tài liệu sau:
{retrieved_chunks}

Trả lời câu hỏi: {user_query}
"""
response = llm.complete(prompt)
```

## Các tool RAG phổ biến

### LangChain
Framework đầy đủ nhất cho RAG pipelines. Có sẵn connectors cho hầu hết vector DBs, loaders cho nhiều định dạng tài liệu.

### LlamaIndex
Tập trung vào document indexing và query. Mạnh hơn LangChain ở phần data ingestion và structured data.

### Qdrant
Vector database hiệu suất cao, hỗ trợ filtering kết hợp với vector search (rất quan trọng cho production).

## Khi nào nên dùng RAG?

✅ Khi có **private knowledge base** (tài liệu nội bộ, docs sản phẩm)
✅ Khi cần **thông tin cập nhật** liên tục
✅ Khi cần **trích dẫn nguồn** cho câu trả lời
✅ Khi **fine-tuning quá tốn kém**

❌ Không cần nếu thông tin đã có trong training data của LLM
❌ Không phù hợp khi cần **reasoning phức tạp** qua nhiều bước (nên dùng Agents)
