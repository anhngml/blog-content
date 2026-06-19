---
title: MCP — Model Context Protocol là gì và tại sao nó quan trọng
description: >-
  Giải thích MCP từ đầu: giao thức chuẩn hóa kết nối AI agents với data sources,
  tại sao Anthropic tạo ra nó và cách nó thay đổi cách xây dựng AI apps
pubDate: 2026-05-15T00:00:00.000Z
tags:
  - MCP
  - AI Agents
  - Anthropic
  - Protocol
lang: vi
concepts:
  - MCP
  - AI Agents
  - Tool Use
  - Function Calling
  - Context Window
tools:
  - Claude
  - Cursor
  - neo4j-mcp
  - filesystem-mcp
topics:
  - AI
  - Agents
relations:
  - to: llm-agents
    type: SAME_TOPIC
  - to: knowledge-graph-ai
    type: SAME_TOPIC
---

## MCP là gì?

Model Context Protocol (MCP) là một **giao thức mở** do Anthropic phát triển (cuối 2024), cho phép AI models kết nối với các data sources và tools theo một cách **chuẩn hóa, thống nhất**.

### Vấn đề trước MCP

Trước MCP, mỗi AI application phải tự build integration riêng:

```
Claude App  → custom connector → Database
Claude App  → custom connector → File system
Claude App  → custom connector → GitHub
Claude App  → custom connector → Jira
...
```

Mỗi connector là code riêng, maintain riêng, bảo mật riêng. **M × N problem** — M apps × N data sources = MN integrations.

### Với MCP

```
Claude App
Cursor                    → MCP Protocol → MCP Server → Database
Any MCP-compatible client             → MCP Server → File system
                                      → MCP Server → GitHub
```

**Một giao thức — Một server — Kết nối với mọi client.**

## Kiến trúc MCP

### 3 thành phần chính

**1. MCP Host** — Ứng dụng AI (Claude Desktop, Cursor, custom app)
**2. MCP Client** — Embedded trong Host, handle protocol
**3. MCP Server** — Expose data/tools theo chuẩn MCP

### MCP Server expose gì?

```typescript
// Tools — LLM có thể gọi
server.tool("search_files", { query: z.string() }, async ({ query }) => {
  const results = await searchFiles(query);
  return { content: [{ type: "text", text: results }] };
});

// Resources — static data sources
server.resource("config://app", async () => {
  return { contents: [{ text: readConfig() }] };
});

// Prompts — pre-built prompt templates
server.prompt("analyze-code", async () => {
  return { messages: [{ role: "user", content: "Analyze this code..." }] };
});
```

## Transport layers

### stdio (local)
Server chạy như subprocess, communicate qua stdin/stdout. Đơn giản nhất, bảo mật nhất vì không expose network.

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./dist/server.js"]
    }
  }
}
```

### Streamable HTTP (remote)
Single endpoint nhận POST và stream SSE responses. Dùng khi cần expose server qua network.

## MCP trong thực tế

### Neo4j MCP Server
```
Claude: "Tìm tất cả posts về RAG"
  → gọi search_blog("RAG")
  → MCP Server query Neo4j
  → trả về danh sách posts
```

### File System MCP
```
Claude: "Đọc và tóm tắt tất cả file .md trong thư mục docs/"
  → gọi list_files("/docs")
  → gọi read_file("/docs/design/TDD.md")
  → gọi read_file("/docs/design/SRS.md")
  → tóm tắt tất cả
```

## Tại sao MCP quan trọng?

MCP đang trở thành **"USB-C của AI integrations"** — một chuẩn duy nhất để kết nối tất cả. Năm 2025, hàng nghìn MCP servers đã được publish, từ Google Drive, GitHub, Slack đến các database như PostgreSQL, Neo4j, Redis.
