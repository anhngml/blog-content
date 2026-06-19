---
title: >-
  Từ brainstorm đến vận hành: vòng đời phát triển phần mềm trong thời
  AI-assisted engineering
description: >-
  Phân tích vòng đời phát triển phần mềm (SDLC) trong thời đại AI-assisted
  engineering: từ brainstorm, spec, tasking, coding đến testing và
  observability.
pubDate: 2026-06-17T00:00:00.000Z
tags:
  - AI-assisted engineering
  - Software Engineering
  - AI Agents
  - SDLC
lang: vi
concepts:
  - AI-assisted Engineering
  - SDLC
  - Context Engineering
  - Task Graph
  - Observability
tools: []
topics:
  - AI-assisted Engineering
  - AI Agents
relations: []
---

Có một hiểu lầm rất phổ biến về AI-assisted engineering: nhiều người nghĩ cuộc chơi là “tìm con agent code giỏi nhất”. Thực tế ngược lại. Tool code giỏi chỉ là một mắt xích. Một hệ thống phát triển phần mềm tốt trong kỷ nguyên AI phải là một **operating system cho engineering**: biến ý tưởng mơ hồ thành spec, biến spec thành plan, biến plan thành task nhỏ, cho agent code trong sandbox/worktree, bắt agent viết test, review bằng nhiều lớp, ship có kiểm soát, rồi ghi lại tri thức vào memory để lần sau không phải dạy lại từ đầu.

Nói ngắn gọn:

```text
Brainstorm
→ Product framing
→ Research & codebase discovery
→ Spec / PRD
→ Architecture & plan
→ Task graph
→ Context engineering
→ Implementation
→ Test & QA
→ Review & security
→ Release
→ Observability / incident learning
→ Memory / knowledge base
→ Metrics & process improvement
```

Stack tốt nhất không phải “Claude Code hay Codex hay Cursor”. Stack tốt nhất là cách ghép:

```text
Spec Kit / OpenSpec / Spec Kitty
+ BMAD / Superpowers / gstack
+ Task Master
+ Claude Code / Codex / Gemini CLI / Copilot / Cline / Roo / Aider / OpenHands
+ Context7 / Repomix / GBrain / AGENTS.md
+ CI / tests / security scanners / code review agents
+ observability / incident workflow
```

Triết lý lõi: **AI được phép tăng tốc, nhưng không được phép làm mờ trách nhiệm engineering.** Requirements vẫn phải rõ. Architecture vẫn phải được review. Tests vẫn phải chạy. Security vẫn phải có gate. Production vẫn phải có rollback. Và con người vẫn phải giữ quyền quyết định cuối.

---

## 1. Giai đoạn brainstorm: biến ý tưởng lỏng thành vấn đề đáng giải

Đây là bước bị nhiều team bỏ qua nhất. Họ mở Cursor, Claude Code hoặc Replit, gõ “build me an app”, rồi sau đó mới phát hiện mình chưa hiểu user, chưa hiểu success metric, chưa hiểu constraint, chưa biết dữ liệu lấy từ đâu.

Brainstorm tốt không phải là tạo thật nhiều idea. Brainstorm tốt là **giảm mơ hồ**.

Artifact cần có sau bước này:

```text
1. Người dùng là ai?
2. Họ đang đau ở đâu?
3. Giải pháp hiện tại là gì?
4. Vì sao bây giờ là thời điểm đúng?
5. Success metric là gì?
6. Non-goals là gì?
7. Những rủi ro lớn nhất là gì?
8. Demo đầu tiên cần chứng minh điều gì?
```

### Công cụ phù hợp

**BMAD Method** rất mạnh cho bước này vì nó đưa AI vào vai trò PM, analyst, architect, UX, dev, QA. Thay vì hỏi một chatbot chung chung, bạn cho từng agent phản biện từ góc nhìn riêng. Với project mới, hãy bắt đầu bằng BMAD analyst/PM để đào requirement, sau đó dùng architect để soi tính khả thi, rồi dùng QA để hỏi “cái gì có thể hỏng?”. [N2]

**Superpowers** hữu ích vì nó ép agent không nhảy ngay vào code. Skill brainstorming của Superpowers hướng agent đi qua đối thoại, làm rõ yêu cầu, rồi mới chuyển sang spec/plan. Đây là cách chống “vibe coding” rất thực tế: AI không được phép hành xử như máy sinh code; nó phải hành xử như engineer biết hỏi lại và biết làm rõ. [N3]

**gstack** phù hợp khi bạn muốn một vòng brainstorm kiểu “AI leadership team”. `/ceo` có thể ép nhìn lại sản phẩm, `/designer` bắt lỗi UX slop, `/eng-manager` soi kiến trúc, `/reviewer` và `/security` phản biện rủi ro. Điểm mạnh của gstack không phải là mỗi skill đều hoàn hảo, mà là nó đóng gói một thói quen đúng: trước khi code, hãy để nhiều vai trò khác nhau tấn công idea. [N5]

**ChatGPT / Claude / Gemini** vẫn rất tốt cho brainstorm tự do, nhưng nên dùng theo form:

```text
Tôi đang định build X cho user Y.
Hãy phỏng vấn tôi như một PM khó tính.
Mỗi lần chỉ hỏi 1–2 câu.
Mục tiêu là biến ý tưởng này thành problem statement, user story, non-goals và success metrics.
Không đề xuất implementation cho tới khi requirement rõ.
```

### Cách dùng hiệu quả

Đừng để AI brainstorm trong khoảng không. Hãy đưa vào: user thật, competitor, constraints, timeline, data source, kỹ năng team, tech stack hiện có. Brainstorm nên kết thúc bằng một **one-page product brief**, không phải một danh sách ý tưởng.

Một template tốt:

```text
# Product Brief

## Problem
## Target users
## Current workaround
## Proposed solution
## Why now
## Success metrics
## Non-goals
## Core flows
## Risks
## Open questions
```

---

## 2. Product framing: từ ý tưởng sang requirement có thể kiểm chứng

Sau brainstorm, việc tiếp theo là biến idea thành requirement. Đây là nơi nhiều AI project hỏng: requirement được viết như “app nên đẹp, nhanh, dễ dùng”. Agent không thể verify những câu đó.

Requirement tốt phải có **acceptance criteria**. Ví dụ:

```text
Không tốt:
Người dùng có thể quản lý task dễ dàng.

Tốt:
Người dùng có thể tạo task với title bắt buộc, due date optional, priority trong {low, medium, high}.
Sau khi tạo, task xuất hiện trong danh sách trong vòng 500ms ở trạng thái optimistic.
Nếu API fail, UI rollback và hiển thị toast lỗi.
```

### Công cụ phù hợp

**GitHub Spec Kit** là công cụ đáng đặt làm xương sống ở giai đoạn này. Nó đại diện cho hướng spec-driven development: specification không còn là tài liệu phụ, mà trở thành artifact trung tâm để agent dựa vào khi generate plan, tasks và implementation. [N1]

**OpenSpec** là lựa chọn khác cho spec-driven workflow, đặc biệt nếu bạn muốn spec trở thành source of truth lâu dài, có proposal, review, approval và custom schema. [N23]

**Spec Kitty** là hướng lightweight hơn: spec → plan → tasks → next → review → accept → merge. Nó đáng xem nếu bạn thích workflow ít nặng hơn Spec Kit/OpenSpec nhưng vẫn muốn ép agent đi theo contract. [N24]

**Task Master** nằm ở biên giữa PRD và tasking: nó parse PRD, phân rã thành task, giữ dependency, cho agent biết “next task” thay vì tự chọn bừa. Đây là tool rất thực dụng để biến product intent thành backlog agent-executable. [N4]

### Cách dùng hiệu quả

Một spec tốt nên có đủ:

```text
# Feature Spec

## User problem
## Goal
## Non-goals
## User stories
## Acceptance criteria
## Data model
## API / interface contract
## Error states
## Permission / auth rules
## Security notes
## Analytics / metrics
## Test scenarios
## Rollout / migration notes
```

Quy tắc vàng: **mỗi requirement phải có cách test**. Nếu không test được, agent sẽ biến nó thành diễn giải chủ quan.

---

## 3. Research & codebase discovery: trước khi sửa repo, phải hiểu repo

AI coding agents thường sai không phải vì chúng không biết syntax. Chúng sai vì thiếu context: không biết convention nội bộ, không biết module nào là source of truth, không biết historical decisions, không biết test nào quan trọng, không biết API nào đã deprecated.

Với repo mới hoặc legacy system, đừng bắt agent code ngay. Bắt nó làm discovery trước.

Artifact cần có:

```text
1. Repo map
2. Main modules
3. Data flow
4. Important commands
5. Test strategy hiện tại
6. Known risky areas
7. Architecture decisions
8. Integration points
9. Gaps / unknowns
```

### Công cụ phù hợp

**Repomix** đóng gói toàn bộ repo thành định dạng thân thiện với LLM. Nó rất hữu ích khi bạn muốn đưa codebase vào Claude/ChatGPT/Gemini để review architecture, tìm coupling, sinh docs hoặc tạo onboarding brief. [N11]

**Sourcegraph / Amp** mạnh ở codebase-scale context. Với hệ thống lớn, vấn đề không phải chỉ là “agent viết code”, mà là agent phải tìm đúng chỗ cần sửa, hiểu dependency và chạy feedback loop. Amp cũng nên được dùng cẩn thận vì agent có thể chạy tool/shell command thay bạn. [N22]

**Reversa** rất đáng chú ý cho legacy code. Nó đi theo hướng reverse documentation engineering: phân tích code hiện có, trích xuất rule ẩn, viết operational specification, đánh dấu confidence và gap để con người validate. Với hệ thống cũ không có docs, đây có thể là bước bắt buộc trước khi để agent sửa code. [N25]

**GBrain** thuộc lớp memory/knowledge graph. Nó không chỉ là RAG trên markdown; triết lý của nó là biến note, meeting, email, people, company, idea thành memory mà agent có thể query và cập nhật. Với team làm sản phẩm dài hạn, đây là mảnh “institutional memory”. [N6]

**Context7** giải quyết vấn đề rất cụ thể: agent hay hallucinate API vì docs trong model cũ. Context7 đưa docs version-specific và code examples vào prompt/MCP để agent dùng đúng API hiện tại. [N10]

### Cách dùng hiệu quả

Trước khi implement feature trong repo lớn, hãy tạo một “discovery ticket”:

```text
Task: Explore how billing plans are implemented today.
Output required:
- Files/modules involved
- Existing data models
- Existing tests
- Existing edge cases
- Similar features to imitate
- Risks
- Recommended implementation plan
Do not edit files.
```

Sau đó mới chuyển sang spec/implementation.

---

## 4. Architecture & design: nơi con người không được abdicate

AI có thể đề xuất architecture, nhưng không nên được giao toàn quyền quyết định architecture. Lý do đơn giản: architecture là quyết định có chi phí dài hạn, liên quan đến organization, team skill, deploy constraints, data ownership, compliance, latency, observability và future roadmap. Agent thường tối ưu cho “giải pháp nghe hợp lý ngay bây giờ”.

Artifact cần có:

```text
1. Architecture overview
2. ADRs
3. Data model
4. API contracts
5. State transitions
6. Failure modes
7. Security model
8. Migration plan
9. Operational model
10. Cost / latency assumptions
```

### Công cụ phù hợp

**BMAD Architect** hữu ích để tạo architecture draft và hỏi các câu mà dev thường bỏ qua: boundary, data ownership, scaling, rollback, observability, security. [N2]

**gstack Eng Manager / Security Officer / Release Engineer** phù hợp để tạo nhiều lượt critique: kiến trúc có bị over-engineer không, có điểm production-risk không, có thiếu migration/rollback không. [N5]

**Spec Kit** nên giữ vai trò nối spec với plan. Sau khi spec được duyệt, plan phải giải thích cách implement và các trade-off. [N1]

**Codex / Claude Code / Gemini CLI / Junie / Amp** có thể dùng để “architecture review pass”, nhưng nên prompt theo hướng review chứ không phải generate:

```text
Review this architecture as a senior backend engineer.
Find:
- hidden coupling
- migration risks
- failure modes
- data consistency problems
- missing tests
- operational risks
Do not propose code yet.
```

### Cách dùng hiệu quả

Mỗi feature lớn nên có ít nhất một ADR:

```text
# ADR: Use event-driven sync for subscription updates

## Context
## Decision
## Alternatives considered
## Consequences
## Rollback strategy
## Observability
```

Với AI-assisted engineering, ADR càng quan trọng hơn vì agent cần biết “vì sao” chứ không chỉ “làm gì”.

---

## 5. Task decomposition: agent cần task nhỏ, rõ, có dependency

Một lỗi phổ biến là giao cho agent task quá lớn:

```text
Build the billing system.
```

Task tốt hơn:

```text
1. Add pricing plan enum and migration.
2. Add subscription table.
3. Add repository methods.
4. Add billing service.
5. Add API endpoint.
6. Add unit tests.
7. Add integration test for upgrade/downgrade.
8. Add UI plan selector.
9. Add e2e happy path.
```

### Công cụ phù hợp

**Task Master** là tool nổi bật ở bước này. Nó đọc PRD, sinh task list, phân tích complexity, dependency, “next task”, và tích hợp với nhiều AI IDE/agent. Điểm mạnh là nó làm agent bớt tự tung tự tác: thay vì “làm gì tiếp?”, agent có một task graph. [N4]

**Spec Kit tasks** phù hợp nếu bạn đã dùng Spec Kit làm workflow chính: spec → plan → tasks → implementation. [N1]

**BMAD stories** hợp với team thích agile/story-driven. BMAD mạnh ở handoff giữa role: PM/PO/architect/dev/QA không nói chuyện bằng prompt rời rạc mà bằng artifact. [N2]

**GitHub Issues / Linear / Jira + MCP** phù hợp khi team đã có project management system. Ở đây, agent không nên thay PM tool; agent nên đọc issue, tạo branch, mở PR, update status, và để con người review.

### Cách dùng hiệu quả

Mỗi task nên có:

```text
# Task

## Goal
## Files likely involved
## Constraints
## Acceptance criteria
## Tests to add/update
## Definition of done
## Out of scope
```

Không giao task kiểu “refactor auth”. Hãy giao:

```text
Refactor auth middleware so that all /api/admin routes require role=admin.
Do not change public routes.
Add tests for:
- unauthenticated request
- authenticated non-admin
- admin user
- existing public route still works
```

---

## 6. Context engineering: repo phải nói chuyện được với agent

Context engineering là phần xương sống của AI-assisted SDLC. Không có context tốt, bạn chỉ đang đánh cược vào trí nhớ model.

Artifact cần có trong repo:

```text
AGENTS.md
CLAUDE.md hoặc GEMINI.md nếu dùng tool-specific config
.cursor/rules nếu dùng Cursor
.github/copilot-instructions.md nếu dùng Copilot
docs/architecture.md
docs/testing.md
docs/deployment.md
docs/decisions/*.md
docs/security.md
```

### Công cụ và chuẩn phù hợp

**AGENTS.md** đang nổi lên như “README cho coding agents”: setup commands, test commands, style guide, repo layout, PR guidelines. Đây là một trong những file có ROI cao nhất vì nhiều agent có thể đọc chung. [N19]

**CLAUDE.md / GEMINI.md / .cursor/rules / copilot-instructions.md** vẫn hữu ích cho tool-specific behavior. Chúng nên tham chiếu đến source of truth chung thay vì copy-paste quá nhiều.

**Claude Code skills** và **Codex skills** cho phép đóng gói workflow thành capability tái sử dụng. Superpowers và gstack thực chất là các skill/command pack có triết lý mạnh. [N8] [N9]

**MCP** là cách chuẩn hóa tool access: agent có thể gọi GitHub, Linear, database, docs, browser, internal tools. Nhưng MCP cũng mở attack surface mới; phải coi mỗi MCP server như một integration có quyền, không phải “plugin vô hại”. [N20]

**llms.txt** hữu ích cho documentation sites: nó giúp agent tìm đúng tài liệu cần đọc thay vì crawl bừa. [N21]

**Context7** nên được dùng cho framework/library thay đổi nhanh. Nó kéo docs đúng version vào ngữ cảnh. [N10]

**Repomix** nên dùng khi cần “snapshot” repo cho review/analysis. [N11]

**GBrain** nên dùng khi tri thức nằm ngoài repo: meeting notes, product decisions, customer conversations, internal docs. [N6]

### Cách dùng hiệu quả

Một `AGENTS.md` tốt nên ngắn, cụ thể, executable:

```text
# AGENTS.md

## Setup
- pnpm install
- pnpm dev

## Test
- pnpm test
- pnpm typecheck
- pnpm lint

## Repo map
- apps/web: Next.js frontend
- packages/db: Prisma schema and migrations
- packages/api: API routes and services

## Rules
- Do not modify migrations after merge.
- Use server actions only in apps/web/app.
- All new API routes need auth tests.
- Never log PII.
- Before PR, run pnpm lint && pnpm test.
```

Đừng biến `AGENTS.md` thành tiểu thuyết. Nếu dài quá, agent sẽ bỏ sót. Hãy link ra docs chi tiết.

---

## 7. Implementation: chọn agent theo loại việc, không theo hype

Không có agent nào tốt nhất cho mọi việc. Cách chọn đúng là dựa vào task shape.

### Nhóm coding agent chính

**Claude Code** mạnh khi cần reasoning sâu, multi-step changes, terminal workflow, GitHub Actions, hooks, skills, subagents. Hooks đặc biệt quan trọng vì chúng cho phép enforce rule deterministic ở các điểm lifecycle: trước/sau tool call, khi stop, khi session start. [N7] [N8]

**OpenAI Codex CLI / Codex app / Codex review** mạnh nếu bạn muốn local terminal agent, app review pane, GitHub code review và `AGENTS.md`-based guidance. Codex code review tập trung vào serious issues và có thể đọc review guidelines trong `AGENTS.md`. [N12] [N13]

**Gemini CLI** là terminal agent open-source của Google. Nó đáng dùng cho team thích terminal, muốn quota tốt, hoặc muốn GitHub Actions cho issue triage/PR review. [N14]

**GitHub Copilot cloud agent** phù hợp cho workflow issue → plan → branch → PR ngay trong GitHub Actions-powered environment. Nó khác agent mode trong IDE: cloud agent làm việc tự động trên GitHub, còn agent mode trong IDE sửa local workspace. [N15]

**Cursor** vẫn rất mạnh cho interactive coding trong IDE, nhất là khi dùng rules, agent mode, codebase context và MCP. Điểm mạnh là UX nhanh, gần dòng code.

**Cline** là open-source coding agent đáng giá vì human-in-the-loop approval, Plan/Act mode, khả năng tạo file, chạy command, browse web, MCP, và cả Kanban/worktree multi-agent workflow. [N16]

**Roo Code** phù hợp với người thích mode-based workflow: Code, Architect, Ask, Debug, Custom Modes. Nó biến editor thành “dev team” nhỏ với nhiều mode khác nhau. [N17]

**Aider** là terminal pair programmer git-native. Nó cực hữu ích cho surgical edits, refactor nhỏ, fix test, commit rõ ràng. Điểm mạnh là đơn giản, ít ceremony, người dùng vẫn steer chặt. [N18]

**OpenHands** phù hợp khi bạn muốn nền tảng open-source cho software engineering agents, sandbox, SDK, self-host hoặc scale nhiều agent. Đây là lựa chọn tốt cho research hoặc enterprise muốn kiểm soát runtime. [N18]

**OpenCode, Continue, Junie, Amp, Zed** là nhóm đáng xem thêm. Continue thú vị ở hướng source-controlled AI checks trên PR. Junie mạnh nếu bạn ở hệ JetBrains. Amp mạnh về agentic terminal/codebase workflow nhưng cần chú ý policy/sandbox. Zed đáng chú ý vì hướng editor nhanh, collaborative, agentic. [N22]

### Chọn agent theo loại task

| Loại việc                     | Tool phù hợp                                                                         |
| ------------------------------- | -------------------------------------------------------------------------------------- |
| Sửa bug nhỏ, edit chính xác | Aider, Codex CLI, Claude Code                                                          |
| Feature vừa có nhiều file    | Claude Code, Codex, Cline, Roo, Cursor                                                 |
| Repo lớn / enterprise          | OpenHands, Sourcegraph/Amp, Copilot cloud agent                                        |
| Issue → PR async               | Copilot cloud agent, Claude Code Action, Gemini CLI GitHub Actions, Codex cloud/review |
| TDD workflow                    | Superpowers + Claude Code/Codex/Aider                                                  |
| UI prototype                    | Cursor, Claude Code, Firebase Studio, Replit, Bolt/Lovable với guardrails             |
| Legacy modernization            | Reversa + Repomix + Spec Kit/OpenSpec + agent implementation                           |
| Review/security pass            | Claude Code Review, Codex Review, CodeRabbit, Qodo, Semgrep, Snyk                      |

### Cách dùng hiệu quả

Implementation nên chạy trong branch/worktree riêng. Mỗi task nên có loop:

```text
1. Read task and acceptance criteria.
2. Inspect relevant files.
3. State implementation plan.
4. Write failing test first where possible.
5. Implement minimal change.
6. Run focused test.
7. Run broader checks.
8. Summarize diff and risks.
```

Prompt tốt:

```text
Implement task 12 only.
Do not modify unrelated files.
First inspect the existing pattern.
Then write or update tests.
Then implement the minimal code change.
Run the targeted tests.
Stop and summarize if tests fail for reasons outside this task.
```

---

## 8. Testing & QA: AI code không có test là nợ kỹ thuật tăng tốc

AI rất giỏi viết code nghe hợp lý. Vấn đề là code nghe hợp lý có thể sai. Vì vậy trong AI-assisted engineering, test không phải phần sau cùng; test là **điều kiện để agent được phép tiếp tục**.

### Công cụ phù hợp

**Superpowers** rất mạnh ở TDD discipline: nó ép red-green-refactor, debugging có phương pháp, implementation plan, code review checkpoint. Đây là tool nên dùng nếu team muốn chống thói quen “agent code trước, test sau”. [N3]

**Qodo / Qodo Cover** phù hợp cho automated test generation, test coverage và review-oriented quality workflows. [N27]

**Codex / Claude / Aider** có thể dùng để tạo test, nhưng phải bắt chúng chạy test và sửa theo kết quả thật.

**Playwright / Cypress** nên dùng cho end-to-end UI flows. Với AI-generated frontend, visual đẹp không có nghĩa flow đúng. E2E test giúp bắt các lỗi “demo chạy được nhưng user path gãy”.

**ABTest-style thinking** đáng học: behavior-driven tests cho agent và workflow, không chỉ test code. Khi agent hay mắc lỗi lặp lại, hãy biến failure đó thành test hoặc checklist. [N31]

### Cách dùng hiệu quả

Đừng prompt:

```text
Add tests.
```

Hãy prompt:

```text
Add tests covering:
- successful creation
- validation error
- unauthorized request
- duplicate submission
- rollback on failed API response
Use existing test patterns in packages/api/__tests__.
Do not mock the unit under test.
Run the focused test file after writing.
```

Mỗi feature nên có “test map”:

```text
Unit tests: business rules
Integration tests: API + DB
E2E tests: user flow
Security tests: auth/permission boundaries
Regression tests: previously broken cases
```

---

## 9. Review: AI review không thay người, nhưng giảm blind spot

Khi agent giúp viết nhiều code hơn, bottleneck chuyển sang review. Nhưng dùng AI review sai cách sẽ tạo noise. Review agent tốt phải có context, rule rõ, severity rõ, và không comment linh tinh.

### Công cụ phù hợp

**Claude Code Review** dùng multi-agent analysis cho PR review sâu, phù hợp với team đã dùng Claude Code và cần review logic/security/regression. [N7]

**Codex code review** hữu ích vì có thể trigger bằng `@codex review`, dùng `AGENTS.md` review guidelines, và tập trung vào serious issues. [N13]

**CodeRabbit** là AI-first PR reviewer với line-by-line suggestions và chat trên PR. Hợp với team muốn review agent chuyên dụng. [N26]

**Qodo Merge / Qodo platform** phù hợp với team muốn code review + quality workflow + governance + custom rules. [N27]

**Continue** đáng chú ý ở hướng “AI checks as repo files”: mỗi check là markdown trong repo, chạy như GitHub status check. Đây là hướng rất đúng về mặt governance: standard nằm trong source control, con người quyết định, AI enforce. [N22]

**Semgrep / Snyk** không nên bị thay bằng LLM reviewer. Chúng là deterministic/static/security scanners, rất cần trong CI. LLM reviewer tốt nhưng không thay SAST/SCA/secrets scanning. [N28]

### Cách dùng hiệu quả

Review agent nên được cấu hình bằng rubric:

```text
Review priorities:
P0: security/data loss/prod outage
P1: correctness/regression/auth bug
P2: missing tests/performance issue
P3: maintainability/style

Do not comment on formatting handled by lint.
Do not suggest large rewrites unless required.
Always cite file/line and explain risk.
Prefer concise comments with patch suggestion.
```

Một PR AI-generated nên có ba lớp:

```text
1. Deterministic checks: lint, typecheck, tests, SAST, secrets
2. AI review: logic, edge cases, missing tests, security reasoning
3. Human review: product intent, architecture, trade-offs, ownership
```

---

## 10. Security: AI làm tăng tốc cũng làm tăng attack surface

AI-assisted engineering có hai loại rủi ro bảo mật:

```text
1. AI sinh code có bug bảo mật.
2. AI tool/agent/hook/MCP/GitHub Action trở thành attack surface mới.
```

Vì vậy security không thể là “review cuối”. Security phải được nhúng vào spec, context, implementation và CI.

### Công cụ phù hợp

**Semgrep**: SAST, SCA, secrets detection, guardrails trong IDE/pre-commit/CI. Rất hợp để enforce rules cụ thể như “không log PII”, “không dùng raw SQL không parameterize”. [N28]

**Snyk / DeepCode AI**: dependency vulnerabilities, code security, container/IaC security, AI-assisted remediation. [N28]

**Claude/Codex/Gemini security review**: tốt cho reasoning pass, threat modeling, auth boundary review, nhưng không thay scanner.

**Gstack Security Officer**: hữu ích như checklist-driven security role trong Claude Code. [N5]

**Constitutional spec-driven development** là hướng đáng học: đưa security constraints vào spec/constitution ngay từ đầu, thay vì chờ code xong mới quét lỗi. [N30]

**MCP security review** là bắt buộc nếu bạn dùng MCP servers. MCP giúp agent gọi tools dễ hơn, nhưng cũng tạo bề mặt tấn công mới: prompt injection, tool poisoning, untrusted server, permission creep. [N20]

### Cách dùng hiệu quả

Trong spec, luôn có section:

```text
## Security requirements
- Auth boundary
- Permission matrix
- Data classification
- PII handling
- Logging rules
- Secrets handling
- Abuse cases
- Rate limits
```

Trong repo, có rule:

```text
Agent must not:
- read or print secrets
- modify production config
- run destructive DB commands
- disable tests
- weaken auth checks
- commit .env files
```

Trong CI, có gate:

```text
lint
typecheck
unit tests
integration tests
secret scan
SAST
dependency scan
IaC scan
container scan
```

---

## 11. Release: agent có thể chuẩn bị release, không nên tự “ném vào production”

AI rất hữu ích cho release notes, changelog, migration checklist, smoke test checklist, rollback plan. Nhưng production deploy cần guardrail.

### Công cụ phù hợp

**gstack Release Engineer** phù hợp để chuẩn bị release checklist, PR summary, changelog, deployment risk. [N5]

**Claude Code GitHub Actions / Gemini CLI GitHub Actions / Copilot cloud agent** có thể tự tạo PR, update issue, triage, review, nhưng merge/deploy nên theo branch protection và approval. [N7] [N14] [N15]

**Codex review** nên chạy trước merge như một high-signal review pass. [N13]

**GitHub Actions** là nơi gắn deterministic gates.

### Cách dùng hiệu quả

Mỗi release nên có:

```text
# Release Checklist

## Changes
## User impact
## DB migrations
## Feature flags
## Rollback plan
## Monitoring dashboards
## Smoke tests
## Known risks
## Owner
```

AI nên được giao:

```text
Generate release notes from merged PRs.
Identify risky changes.
Draft rollback plan.
List smoke tests.
Do not deploy.
```

---

## 12. Observability & incident learning: vòng đời không kết thúc ở merge

Sau release, software bắt đầu nói sự thật. Logs, traces, metrics, errors, support tickets, user feedback — đây là dữ liệu để cải thiện spec và memory.

### Công cụ phù hợp

**OpenTelemetry** nên là baseline nếu bạn muốn telemetry vendor-neutral: traces, metrics, logs, export sang backend khác nhau. [N29]

**Sentry** phù hợp cho error tracking, performance monitoring, issue grouping, debugging feedback loop. [N29]

**Datadog / PagerDuty** phù hợp cho observability, on-call, incident management, digital operations. [N29]

**GBrain** hoặc internal knowledge base nên ingest postmortems, incident notes, customer support themes, product decisions. Nếu không, team sẽ lặp lại lỗi cũ. [N6]

### Cách dùng hiệu quả

Mỗi incident nên tạo artifact:

```text
# Incident Note

## What happened
## Customer impact
## Timeline
## Root cause
## Detection gap
## Prevention
## Follow-up tasks
## Spec/docs updates needed
```

Sau postmortem, bắt agent:

```text
Update docs/decisions and AGENTS.md if this incident revealed a rule future agents must follow.
Create regression tests for the root cause.
Open follow-up issues.
```

---

## 13. Memory & company brain: agent giỏi hơn khi tổ chức biết nhớ

Context window không phải memory. RAG không tự động là memory. Memory tốt phải có cấu trúc, provenance, update rule, expiry, ownership.

### Công cụ phù hợp

**GBrain** đáng chú ý vì nó định vị memory như production brain cho agents: ingest nhiều nguồn, enrich entities, dùng graph/vector search, sửa citation, consolidate memory. [N6]

**AGENTS.md / docs/decisions / architecture docs** là memory tối thiểu nằm trong repo. Đây là memory rẻ nhất và dễ review nhất.

**Repomix** tạo snapshot context, nhưng không phải memory sống.

**Context7** là memory/docs bên ngoài cho library/framework.

**Sentry/Datadog/PagerDuty postmortem + docs update** là memory vận hành.

### Cách dùng hiệu quả

Hãy phân loại memory:

```text
Repo memory:
- architecture
- conventions
- commands
- tests
- decisions

Product memory:
- users
- roadmap
- customer feedback
- product principles

Operational memory:
- incidents
- runbooks
- dashboards
- alerts
- rollback steps

Agent memory:
- recurring mistakes
- preferred patterns
- successful prompts/workflows
```

Quy tắc: memory chỉ hữu ích nếu có review. Đừng để agent tự ghi memory không kiểm soát rồi lần sau coi đó là sự thật tuyệt đối.

---

## 14. Tool map toàn vòng đời

| Giai đoạn        | Tool nên ưu tiên                                                                       | Vai trò                                 |
| ------------------ | ----------------------------------------------------------------------------------------- | ---------------------------------------- |
| Brainstorm         | BMAD, Superpowers, gstack, ChatGPT/Claude/Gemini                                          | Làm rõ problem, user, non-goals, risks |
| Product brief      | BMAD PM/Analyst, Spec Kit, OpenSpec                                                       | Biến idea thành requirement            |
| Codebase discovery | Repomix, Reversa, Sourcegraph, GBrain, Context7                                           | Hiểu repo/docs/memory trước khi sửa  |
| Spec               | Spec Kit, OpenSpec, Spec Kitty                                                            | Source of truth cho feature              |
| Architecture       | BMAD Architect, gstack Eng Manager, Claude/Codex review                                   | ADR, data flow, trade-off                |
| Tasking            | Task Master, Spec Kit tasks, GitHub Issues/Linear/Jira                                    | Task graph, dependency, next task        |
| Context            | AGENTS.md, CLAUDE.md, GEMINI.md, Cursor rules, Context7, GBrain                           | Cấp đúng context cho agent            |
| Implementation     | Claude Code, Codex, Gemini CLI, Copilot, Cursor, Cline, Roo, Aider, OpenHands, Junie, Amp | Code/test/refactor/debug                 |
| TDD/QA             | Superpowers, Playwright, Cypress, Qodo Cover, Codex/Claude                                | Test-first, regression, e2e              |
| Review             | Codex Review, Claude Code Review, CodeRabbit, Qodo, Continue                              | PR review, quality checks                |
| Security           | Semgrep, Snyk, gstack security, LLM threat modeling                                       | SAST/SCA/secrets/security reasoning      |
| Release            | GitHub Actions, gstack release, Claude/Gemini/Copilot Actions                             | Changelog, checklist, release PR         |
| Observability      | OpenTelemetry, Sentry, Datadog, PagerDuty                                                 | Production feedback                      |
| Memory             | GBrain, docs/decisions, AGENTS.md, postmortems                                            | Institutional learning                   |

---

## 15. Một workflow mẫu cực thực dụng

Đây là workflow mình sẽ áp dụng cho một team nhỏ 3–8 người:

```text
1. Brainstorm bằng BMAD + Superpowers
2. Viết Product Brief
3. Tạo spec bằng Spec Kit
4. Dùng BMAD Architect + gstack Eng Manager review spec
5. Dùng Task Master phân rã task
6. Update AGENTS.md / docs/architecture nếu thiếu context
7. Với mỗi task:
   - agent tạo worktree/branch
   - inspect code
   - viết test trước nếu phù hợp
   - implement minimal diff
   - chạy lint/typecheck/test
8. Mở PR
9. Chạy CI + Semgrep/Snyk
10. Chạy Codex Review hoặc Claude Code Review
11. Human review
12. Merge sau khi pass gates
13. Release notes bằng gstack/Codex/Claude
14. Sau release, theo dõi Sentry/Datadog
15. Nếu có incident, update tests + docs + memory
```

Stack cụ thể:

```text
Core methodology:
- Spec Kit
- BMAD
- Superpowers

Tasking:
- Task Master

Coding:
- Claude Code hoặc Codex CLI
- Aider cho edit nhỏ
- Cline/Roo nếu muốn open-source IDE agent

Context:
- AGENTS.md
- Context7
- Repomix
- GBrain nếu có knowledge base dài hạn

Review/security:
- Codex Review hoặc Claude Code Review
- CodeRabbit hoặc Qodo nếu muốn dedicated PR reviewer
- Semgrep + Snyk trong CI

Ops:
- OpenTelemetry
- Sentry
- Datadog/PagerDuty tùy scale
```

---

## 16. Sai lầm thường gặp

### Sai lầm 1: dùng agent để thay spec

Prompt “build me X” là spec tệ. Agent sẽ tự lấp khoảng trống bằng assumptions. Những assumptions đó trở thành code.

### Sai lầm 2: context quá dài nhưng không có cấu trúc

Nhồi 200k token vào agent không bằng một `AGENTS.md` rõ, một spec tốt, một task nhỏ và một test cụ thể.

### Sai lầm 3: để agent sửa quá nhiều file

PR AI-generated càng lớn càng khó review. Hãy ép task nhỏ, branch nhỏ, diff nhỏ.

### Sai lầm 4: review bằng AI nhưng không có rubric

AI reviewer không có rubric sẽ comment lan man. Hãy nói rõ severity, scope, điều gì không cần comment.

### Sai lầm 5: dùng MCP như plugin vô hại

MCP server có thể có quyền đọc/ghi hệ thống. Hãy treat MCP như production integration: least privilege, allowlist, audit, sandbox.

### Sai lầm 6: không ghi memory sau incident

Nếu agent gây lỗi và team chỉ sửa code mà không update docs/tests/rules, lỗi sẽ quay lại.

---

## 17. Nguyên tắc chọn tool

Không chọn tool vì hype. Chọn theo câu hỏi:

```text
1. Tool này tạo artifact bền vững hay chỉ sinh output tạm?
2. Nó có giúp giảm ambiguity không?
3. Nó có tích hợp được với repo/CI/review không?
4. Nó có chạy trong sandbox/worktree không?
5. Nó có permission model rõ không?
6. Nó có hỗ trợ human approval không?
7. Nó có làm context tốt hơn không?
8. Nó có giúp test/review/security tốt hơn không?
9. Nó có bị lock-in không?
10. Có thể đo hiệu quả bằng metric nào?
```

Metric nên đo:

```text
- cycle time
- PR size
- test pass rate
- review comments per PR
- rollback rate
- escaped defects
- security findings
- incident count
- time-to-restore
- rework rate
- agent cost per merged PR
```

Đừng chỉ đo “code viết nhanh hơn”. AI có thể viết nhanh hơn nhưng làm review, debug, rollback chậm hơn.

---

## 18. Kết luận: AI-assisted engineering là process design, không phải prompt magic

Làn sóng coding agent đang đẩy software engineering từ “AI autocomplete” sang “AI collaborator”. Nhưng collaborator tốt cần luật chơi. Những repo/framework tốt nhất hiện nay đều đi về cùng một hướng:

```text
Spec Kit = định nghĩa cái cần build
BMAD = tổ chức team AI để nghĩ/build/review
Superpowers = ép agent hành xử có kỷ luật
Task Master = biến PRD thành task graph
Context7 / Repomix / GBrain = cấp đúng context và memory
Claude Code / Codex / Gemini / Cline / Roo / Aider / OpenHands = thực thi
Semgrep / Snyk / CodeRabbit / Qodo / Continue = kiểm tra và review
OpenTelemetry / Sentry / Datadog / PagerDuty = học từ production
```

Cách dùng AI tốt nhất không phải là “tin agent hơn”. Cách dùng AI tốt nhất là thiết kế một hệ thống mà agent khó làm sai: spec rõ, task nhỏ, context đúng, tests bắt buộc, review nhiều lớp, security gate, release có rollback, và memory được cập nhật sau mỗi bài học.

AI-assisted engineering trưởng thành không giống vibe coding. Nó giống một nhà máy phần mềm có kỷ luật — trong đó AI là lực lượng tăng tốc, còn methodology là đường ray.

## Nguồn web chính đã dùng

* GitHub Spec Kit và spec-driven development: repo chính thức của Spec Kit mô tả SDD như cách biến specification thành artifact trung tâm, còn bài GitHub blog nói Spec Kit hoạt động với Copilot, Claude Code và Gemini CLI qua các command điều hướng agent. ([GitHub][1])
* BMAD Method: docs chính thức mô tả BMAD là AI-driven development framework đi từ ideation/planning tới agentic implementation, với specialized agents và planning thích ứng độ phức tạp. ([BMAD Method][2])
* Superpowers: repo chính thức mô tả đây là software development methodology cho coding agents dựa trên composable skills và bootstrap instructions để agent tự dùng skills khi cần. ([GitHub][3])
* Task Master: repo `eyaltoledano/claude-task-master` mô tả task management system cho AI-driven development, hoạt động với nhiều AI chat/editor. ([GitHub][4])
* gstack và GBrain: gstack mô tả Claude Code setup thành virtual engineering team với 23 specialists; GBrain mô tả memory/knowledge system dùng cho agents với pages, people, companies và cron jobs. ([GitHub][5])
* Claude Code: docs chính thức về GitHub Actions, hooks, skills và code review cho thấy Claude Code có workflow `@claude`, lifecycle hooks, modular skills và automated PR review. ([Claude Code][6])
* OpenAI Codex: docs chính thức mô tả Codex CLI là local coding agent, Codex review dùng `AGENTS.md`, và agent skills là reusable workflows/packages. ([OpenAI Developers][7])
* Gemini CLI: repo chính thức mô tả Gemini CLI là open-source AI agent trong terminal; Google blog mô tả Gemini CLI GitHub Actions beta cho issue triage và PR review. ([GitHub][8])
* Copilot cloud agent: GitHub docs mô tả cloud agent làm việc trong GitHub Actions-powered environment để research repo, plan, change code và mở PR; GitHub cũng có file custom instructions cho Copilot. ([GitHub Docs][9])
* Context, repo packing và memory: Context7 repo, Repomix repo/site và AGENTS.md standard lần lượt đại diện cho docs context, codebase packing và repo-level agent instructions. ([GitHub][10])
* Coding agents open-source và IDE agents: Cline, Roo Code, Aider, OpenHands, OpenCode, Continue, Junie, Amp và Zed đều có docs/repo chính thức mô tả các hướng triển khai khác nhau của agentic coding. ([GitHub][11])
* Spec-driven alternatives và legacy workflows: OpenSpec, Spec Kitty và Reversa là các hướng bổ sung cho spec-driven/lightweight spec/reverse documentation engineering. ([GitHub][12])
* Review/security tools: CodeRabbit, Qodo, Semgrep và Snyk là các lớp review/security/quality khác nhau, từ AI PR review tới SAST/SCA/secrets/dependency scanning. ([CodeRabbit][13])
* Observability/operations: OpenTelemetry là vendor-neutral observability framework; Sentry, Datadog và PagerDuty đại diện cho error tracking, monitoring và incident management. ([OpenTelemetry][14])
* Bằng chứng và cảnh báo thực tế: các paper 2025–2026 về AI coding agents, AGENTS.md, MCP security, AI code review, Reversa, Spec Kit Agents và agent workflow injection cho thấy adoption tăng nhanh nhưng rủi ro context, review, bảo mật và workflow injection là rất thật. ([arXiv][15])

[1]: https://github.com/github/spec-kit?utm_source=chatgpt.com
[2]: https://docs.bmad-method.org/?utm_source=chatgpt.com
[3]: https://github.com/obra/superpowers?utm_source=chatgpt.com
[4]: https://github.com/eyaltoledano/claude-task-master?utm_source=chatgpt.com
[5]: https://github.com/garrytan/gstack?utm_source=chatgpt.com
[6]: https://code.claude.com/docs/en/github-actions?utm_source=chatgpt.com
[7]: https://developers.openai.com/codex/cli?utm_source=chatgpt.com
[8]: https://github.com/google-gemini/gemini-cli?utm_source=chatgpt.com
[9]: https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent?utm_source=chatgpt.com
[10]: https://github.com/yamadashy/repomix?utm_source=chatgpt.com
[11]: https://github.com/cline/cline?utm_source=chatgpt.com
[12]: https://github.com/Fission-AI/OpenSpec/?utm_source=chatgpt.com
[13]: https://coderabbit.ai/?utm_source=chatgpt.com
[14]: https://opentelemetry.io/docs/?utm_source=chatgpt.com
[15]: https://arxiv.org/abs/2602.09185?utm_source=chatgpt.com

---

## CodeGraph vs Serena: Có **cùng vùng mục đích**, nhưng **không cùng trọng tâm**.

Nếu bạn đang nói tới **`colbymchenry/codegraph`**, thì cả CodeGraph và Serena đều giải quyết một vấn đề chung: giúp AI coding agent **hiểu codebase bằng cấu trúc/symbol thay vì grep + read file mù**. CodeGraph tự mô tả là “pre-indexed knowledge graph” cho agents, gồm symbol relationships, call graphs và code structure để agent query nhanh hơn thay vì scan file bằng grep/glob/read. Serena thì tự mô tả là “IDE for your coding agent”, cung cấp semantic code retrieval, editing, refactoring và debugging ở cấp symbol qua MCP. ([GitHub][1])

Nhưng khác biệt chính là:

```text
CodeGraph = bản đồ / knowledge graph của codebase
Serena    = IDE semantic toolkit cho agent
```

## So sánh nhanh

| Tiêu chí           | Serena                                                                                                            | CodeGraph                                                                                                       |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Mục tiêu chính     | Cho agent năng lực giống IDE: tìm symbol, reference, edit/refactor/debug                                          | Cho agent một knowledge graph đã index sẵn: symbol graph, call graph, impact/context query                      |
| Trọng tâm          | **Semantic operations**: navigate + edit/refactor an toàn                                                         | **Codebase map**: query graph nhanh, giảm token/tool calls                                                      |
| Backend            | Language Server Protocol hoặc JetBrains backend                                                                   | Pre-indexed graph, thường dựa trên parsing/indexing; repo mô tả symbol relationships/call graphs/code structure |
| MCP                | Có, là MCP toolkit                                                                                                | Có, expose graph/context cho AI assistants                                                                      |
| Editing/refactor   | Mạnh: replace symbol body, rename symbol, safe delete, insert before/after symbol; JetBrains backend còn mạnh hơn | Trọng tâm không phải editing; chủ yếu retrieval/context/impact                                                  |
| Debugging          | Có, đặc biệt qua JetBrains plugin: breakpoint, inspect variable, evaluate expression                              | Không phải điểm chính                                                                                           |
| Use case mạnh nhất | Refactor đa file, sửa symbol cụ thể, navigation chính xác, debug                                                  | Hiểu kiến trúc, call graph, impact analysis, build task-specific context                                        |
| Cách nghĩ          | “Agent có tay và mắt của IDE”                                                                                     | “Agent có Google Maps của codebase”                                                                             |

## Cách phân biệt rất thực tế

Khi bạn hỏi:

```text
Hàm này nằm ở đâu?
Ai gọi nó?
Nó gọi những gì?
Nếu sửa symbol này thì ảnh hưởng vùng nào?
Context liên quan tới task này gồm những phần nào?
```

→ **CodeGraph rất hợp**.

Khi bạn hỏi:

```text
Rename symbol này an toàn toàn repo.
Thay body function này.
Insert helper sau class này.
Safe-delete symbol nếu không còn reference.
Tìm diagnostics của file/symbol.
Debug bằng breakpoint.
```

→ **Serena hợp hơn**.

Serena có retrieval giống CodeGraph ở một phần, nhưng nó đi xa hơn ở **symbolic editing/refactoring/debugging**. CodeGraph có thể có context/impact analysis tốt hơn theo hướng graph-native, nhưng không nên coi nó là IDE refactoring engine.

## Nên dùng cái nào?

Với repo nhỏ hoặc task đơn giản, dùng một trong hai có thể đã đủ.

Với repo lớn, mình sẽ không coi chúng là thay thế nhau. Mình sẽ ghép như sau:

```text
CodeGraph
→ map codebase, call graph, impact analysis, task-specific context

Serena
→ thao tác semantic chính xác: find references, rename, replace symbol body, safe delete, diagnostics/debug

Claude Code / Codex / Cline / Roo / Cursor
→ agent reasoning và execution
```

Một workflow đẹp:

```text
1. Dùng CodeGraph để hỏi “task này chạm vào vùng code nào?”
2. Dùng Serena để inspect symbol cụ thể và sửa/refactor an toàn.
3. Dùng agent chính để chạy test, review diff, mở PR.
```

## Verdict

**Cùng mục đích cấp cao:** giúp agent hiểu codebase tốt hơn và giảm grep/read-file/token waste.
**Khác mục đích cấp thấp:** CodeGraph thiên về **knowledge graph / context retrieval / impact map**; Serena thiên về **IDE-grade semantic navigation + editing + refactoring + debugging**.

Nếu phải nói bằng một câu: **CodeGraph giúp agent biết nên nhìn vào đâu; Serena giúp agent sửa đúng chỗ một cách an toàn hơn.**

[1]: https://github.com/colbymchenry/codegraph?utm_source=chatgpt.com "colbymchenry/codegraph: Pre-indexed code knowledge ..."
