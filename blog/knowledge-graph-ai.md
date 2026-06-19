---
title: Knowledge Graph trong AI — Tại sao Graph Database là tương lai của AI Memory
description: >-
  Khám phá tại sao knowledge graph đang thay thế vector-only RAG: structured
  relationships, temporal reasoning, và explainability mà vector DB không có
pubDate: 2026-05-18T00:00:00.000Z
tags:
  - Knowledge Graph
  - Neo4j
  - GraphRAG
  - AI Memory
  - Semantic Web
lang: vi
concepts:
  - Knowledge Graph
  - GraphRAG
  - Semantic Relationships
  - Entity Extraction
  - Temporal Reasoning
  - Explainability
tools:
  - Neo4j
  - Graphiti
  - Memgraph
  - Weaviate
  - LangChain
topics:
  - AI
  - Knowledge Graph
relations:
  - to: vector-databases
    type: SAME_TOPIC
  - to: mcp-introduction
    type: SAME_TOPIC
  - to: rag-explained
    type: SAME_TOPIC
---

## Tại sao Vector DB chưa đủ?

Vector databases xuất sắc ở **semantic similarity search** — tìm những gì "có nghĩa gần giống nhau". Nhưng chúng mù quáng với **relationships** — mối quan hệ giữa các entities.

### Ví dụ minh họa

Câu hỏi: *"Những bài viết nào đề cập đến RAG VÀ được viết sau khi GPT-4 ra mắt AND liên quan đến Qdrant?"*

**Vector DB:** Không thể trả lời trực tiếp — phải post-filter thủ công.

**Knowledge Graph (Cypher):**
```cypher
MATCH (post:Post)-[:MENTIONS]->(c:Concept {name: "RAG"})
WHERE post.pubDate > date("2023-03-14")
AND (post)-[:USES]->(:Tool {name: "Qdrant"})
RETURN post.title, post.pubDate
ORDER BY post.pubDate DESC
```

Đây là sức mạnh của **structured relationships**.

## Knowledge Graph là gì?

Một knowledge graph là mạng lưới **entities (nodes)** và **relationships (edges)**:

```
(GPT-4) -[COMPETES_WITH]-> (Claude 3)
(RAG)   -[REQUIRES]->      (Vector DB)
(Qdrant) -[IMPLEMENTS]->   (Vector DB)
(LangChain) -[USED_FOR]->  (Agents)
```

Mỗi node và edge có thể có **properties** (metadata) và **temporal validity** (từ khi nào đến khi nào).

## GraphRAG — Kết hợp Graph + Vector

Microsoft Research giới thiệu GraphRAG (2024): thay vì chỉ tìm chunks tương tự, GraphRAG:

1. **Extract** entities và relationships từ documents
2. **Build** community clusters (topic groups)
3. **Generate** summaries cho mỗi community
4. **Query** theo 2 modes: Local (specific) và Global (overview)

```
Traditional RAG: "Tìm chunks giống query"
GraphRAG:        "Hiểu cấu trúc của toàn bộ corpus,
                  trả lời ở cả micro và macro level"
```

## Neo4j cho AI Blog

Với blog cá nhân, Neo4j cho phép:

```cypher
-- Tìm tất cả bài liên quan đến RAG (2 hops)
MATCH (p:Post)-[:MENTIONS|USES*1..2]-(related)
WHERE (p)-[:MENTIONS]->(:Concept {name: "RAG"})
RETURN DISTINCT related

-- Mindmap: AI > RAG > Concepts
MATCH (root:Topic {name: "AI"})-[:HAS_SUBTOPIC*1..3]->(topic:Topic)
OPTIONAL MATCH (topic)-[:HAS_CONCEPT]->(c:Concept)
RETURN root, topic, c

-- Semantic search + graph traversal (hybrid)
CALL db.index.vector.queryNodes('post_embeddings', 10, $queryVector)
YIELD node AS post, score
MATCH (post)-[:MENTIONS]->(concept:Concept)
RETURN post.title, collect(concept.name), score
```

## Graphiti — Temporal Knowledge Graph

[Graphiti](https://github.com/getzep/graphiti) là thư viện Python tự động build temporal knowledge graph:

```python
from graphiti_core import Graphiti

g = Graphiti(neo4j_uri, neo4j_user, neo4j_pass)

# Ingest một bài viết như một "episode"
await g.add_episode(
    name="RAG Explained",
    episode_body=post_content,
    source_description="Blog post về RAG"
)

# Graph tự động được cập nhật với entities và relationships
# Graphiti track THỜI GIAN của mỗi fact
```

**Temporal awareness:** Graphiti biết "Tool X được recommend *trước đây*, nhưng Tool Y *hiện tại* tốt hơn."

## Knowledge Graph vs Vector DB — Khi nào dùng gì?

| Use case | Vector DB | Knowledge Graph |
|----------|-----------|-----------------|
| Semantic similarity | ✅ Tốt nhất | ❌ |
| Relationship queries | ❌ | ✅ Tốt nhất |
| Temporal reasoning | ❌ | ✅ |
| Explainability | ❌ | ✅ |
| Recommendation | Partial | ✅ |
| Simple RAG | ✅ | Overkill |

**Kết luận:** Dùng cả hai! Vector DB cho semantic search, Knowledge Graph cho structured reasoning. Đây chính là **GraphRAG** approach.
