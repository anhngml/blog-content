---
title: 'So sánh Vector Databases: Qdrant vs ChromaDB vs Weaviate vs Pinecone'
description: >-
  So sánh toàn diện các vector database phổ biến nhất 2026: performance,
  features, pricing, và khi nào nên chọn cái nào
pubDate: 2026-05-12T00:00:00.000Z
tags:
  - Vector DB
  - RAG
  - Database
  - AI Infrastructure
lang: vi
concepts:
  - Vector DB
  - Vector Database
  - HNSW
  - ANN Search
  - Approximate Nearest Neighbor (ANN) Search
  - Embedding
  - Hybrid Search
  - Filtering
tools:
  - Qdrant
  - ChromaDB
  - Weaviate
  - Pinecone
  - Milvus
  - pgvector
topics:
  - Vector Databases
relations:
  - to: rag-explained
    type: SEQUEL
  - to: knowledge-graph-ai
    type: SAME_TOPIC
---

## Tại sao cần Vector Database?

Regular databases lưu dữ liệu có cấu trúc (số, text, date). Vector databases lưu **embeddings** — các vector hàng nghìn chiều đại diện cho ngữ nghĩa của văn bản, hình ảnh, âm thanh.

Câu hỏi quan trọng nhất trong vector DB: **"Tìm K vectors gần nhất với query vector"** — gọi là **Approximate Nearest Neighbor (ANN) search**.

## So sánh chi tiết

### Qdrant — Khuyến nghị cho production

**Viết bằng:** Rust → performance cực cao, memory safe

```python
from qdrant_client import QdrantClient
client = QdrantClient("localhost", port=6333)

# Tạo collection
client.create_collection("blog_posts", vectors_config=VectorParams(
    size=1536, distance=Distance.COSINE
))

# Upsert vectors
client.upsert("blog_posts", points=[
    PointStruct(id=1, vector=[...], payload={"slug": "rag-explained"})
])

# Search với filter
client.search("blog_posts",
    query_vector=[...],
    query_filter=Filter(must=[FieldCondition(key="lang", match=MatchValue(value="vi"))]),
    limit=5
)
```

**Strengths:**
- Filtering rất mạnh (kết hợp vector + metadata filter)
- Sparse + dense hybrid search
- Self-hosted hoặc cloud
- REST + gRPC APIs

### ChromaDB — Tốt nhất cho prototype

```python
import chromadb
client = chromadb.Client()
collection = client.create_collection("blog_posts")

collection.add(
    documents=["RAG explained...", "MCP introduction..."],
    ids=["rag-explained", "mcp-introduction"]
)

results = collection.query(query_texts=["vector search"], n_results=3)
```

**Strengths:**
- Cực dễ dùng, zero config
- Tự động embed documents (dùng local model)
- Perfect cho local dev và prototyping

**Weaknesses:**
- Performance không tốt với large datasets (>1M vectors)

### Weaviate — Best for hybrid search

```graphql
{
  Get {
    BlogPost(
      hybrid: {
        query: "vector database comparison"
        alpha: 0.75  # 0=BM25, 1=vector
      }
      limit: 5
    ) { title slug description }
  }
}
```

**Strengths:**
- Hybrid search (BM25 + vector) out-of-the-box
- GraphQL API
- Auto-vectorization (OpenAI, Cohere, local models)

### Pinecone — Managed cloud chuyên biệt

Fully managed, zero infra management. Tốt nhất nếu không muốn tự host.

**Pricing:** $0.08/1M reads (khá đắt ở scale lớn)

### pgvector — Nếu đã dùng PostgreSQL

```sql
CREATE EXTENSION vector;
ALTER TABLE posts ADD COLUMN embedding vector(1536);

-- Search
SELECT slug, title, 1 - (embedding <=> '[...]') AS similarity
FROM posts
ORDER BY embedding <=> '[...]'
LIMIT 5;
```

**Best for:** Đã có PostgreSQL, không muốn thêm infra mới

## Bảng so sánh nhanh

| Feature | Qdrant | ChromaDB | Weaviate | Pinecone | pgvector |
|---------|--------|----------|----------|----------|----------|
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Hybrid search | ✅ | ❌ | ✅ | ✅ | Partial |
| Self-hosted | ✅ | ✅ | ✅ | ❌ | ✅ |
| Ease of use | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Best for | Production | Prototype | Hybrid | Managed | Existing PG |

## Recommendation

- **Prototype/học:** ChromaDB
- **Production self-hosted:** Qdrant
- **Không muốn manage infra:** Pinecone
- **Cần hybrid search mạnh:** Weaviate
- **Đang dùng PostgreSQL:** pgvector
