---
title: 'Live Content Collection: Ket Thuc Ky Nguyen Rebuild'
description: >-
  Astro 6 Live Content Collection giup bai viet xuat hien ngay lap tuc sau khi
  dang, khong can rebuild Docker.
pubDate: '2026-06-19'
lang: vi
tags:
  - Astro
  - Live Collection
  - MCP
concepts:
  - Live Content Collection
  - mtime cache
tools: []
topics: []
---
## Van de

Tuong la `blog_publish` chi can ghi file la xong. Nhung khong - Astro 5+ bake content vao JS bundle luc build. File moi ghi vao volume khong duoc Astro biet den cho den khi Docker rebuild.

## Giai phap: Live Content Collection

Astro 6 ho tro chinh thuc **Live Content Collection** - doc du lieu tai **request time** thay vi build time.

```typescript
// src/live.config.ts
const blog = defineLiveCollection({
  loader: fsBlogLiveLoader('./src/content/blog'),
  schema: z.object({ ... }),
});
```

### Cach hoat dong

1. Agent goi `blog_publish` qua MCP
2. File markdown duoc ghi vao content volume
3. Request tiep theo den `/blog/[slug]` trigger `loadEntry()`
4. Loader kiem tra `mtime` - neu thay doi, render lai markdown voi Shiki
5. Neu khong doi, tra tu cache (< 1ms)

![Live Collection Flow](/images/live-collection-flow.png)

## Ket luan

Bai viet nay duoc dang hoan toan qua MCP, co anh, co code block, va xuat hien ngay tren site.

```python
import datetime
print(f"Published at {datetime.datetime.now()}")
```

**Khong rebuild. Khong restart. Live.**
