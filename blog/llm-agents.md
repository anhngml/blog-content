---
title: Xây dựng AI Agents với LLM Tool Use — Từ đầu đến production
description: >-
  Hướng dẫn toàn diện xây dựng AI Agents: tool use, planning, memory,
  multi-agent systems và những pitfalls cần tránh
pubDate: 2026-05-08T00:00:00.000Z
tags:
  - AI Agents
  - LLM
  - Tool Use
  - MCP
  - LangChain
lang: vi
concepts:
  - AI Agents
  - Tool Use
  - Function Calling
  - Planning
  - ReAct
  - Memory
  - Multi-agent
tools:
  - LangChain
  - LangGraph
  - OpenAI
  - Claude
  - AutoGen
  - CrewAI
topics:
  - AI
  - Agents
relations:
  - to: mcp-introduction
    type: SAME_TOPIC
  - to: rag-explained
    type: SAME_TOPIC
---

## AI Agent là gì?

Trong khi LLM đơn thuần chỉ **nhận input → sinh output**, một AI Agent có thể:

1. **Lập kế hoạch** — chia task thành các bước nhỏ
2. **Sử dụng tools** — gọi functions, APIs, databases
3. **Quan sát kết quả** — đọc output của tools
4. **Điều chỉnh** — thay đổi plan dựa trên observation
5. **Lặp lại** — cho đến khi hoàn thành task

### ReAct Pattern (Reasoning + Acting)

```
Thought: Tôi cần tìm thông tin về RAG
Action: search_web("RAG retrieval augmented generation")
Observation: RAG là kỹ thuật kết hợp retrieval với LLM...

Thought: Tôi cần code ví dụ
Action: search_code("RAG python example langchain")
Observation: ```python ...

Thought: Đã đủ thông tin, viết câu trả lời
Final Answer: RAG là...
```

## Tool Use / Function Calling

### OpenAI Function Calling

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_blog",
            "description": "Tìm kiếm bài viết theo semantic query",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"},
                    "limit": {"type": "integer", "default": 5}
                },
                "required": ["query"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Tìm bài về RAG"}],
    tools=tools,
    tool_choice="auto"
)

# Check if LLM wants to call a tool
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    # Execute the tool
    result = search_blog(**json.loads(tool_call.function.arguments))
    # Feed result back to LLM
```

## Memory trong Agents

### 4 loại memory

**1. In-context (Conversation history)**
Đơn giản nhất: giữ messages trong context window. Giới hạn bởi context length.

**2. External (Vector DB)**
```python
# Lưu memory
memory.add("User thích Python hơn JavaScript")

# Retrieve khi cần
relevant = memory.search("user preference programming language")
```

**3. Knowledge Graph (Graphiti)**
Structured memory với temporal awareness. Biết "User thích Python *trước đây*, hiện tại thích Rust".

**4. Episodic (Summarization)**
Tóm tắt conversations cũ, giữ essence mà không giữ toàn bộ text.

## Multi-agent Systems

### Khi nào cần multi-agent?

- Task quá phức tạp cho 1 agent
- Cần parallel execution
- Muốn specialize agents cho từng domain

### LangGraph Example

```python
from langgraph.graph import StateGraph

workflow = StateGraph(AgentState)
workflow.add_node("researcher", research_agent)
workflow.add_node("writer", writing_agent)
workflow.add_node("reviewer", review_agent)

workflow.add_edge("researcher", "writer")
workflow.add_edge("writer", "reviewer")
workflow.add_conditional_edges("reviewer",
    lambda s: "rewrite" if s["needs_revision"] else "end"
)
```

## Pitfalls cần tránh

### 1. Infinite loops
Agent gọi tool → tool fail → gọi lại → loop mãi. Cần max_iterations limit.

### 2. Tool overuse
LLM có xu hướng gọi tools kể cả khi không cần. Cần clear tool descriptions.

### 3. Hallucinated tool calls
LLM "bịa" arguments cho tools. Cần schema validation nghiêm ngặt.

### 4. Cost explosion
Nhiều tool calls = nhiều LLM calls = nhiều tiền. Monitor carefully.
