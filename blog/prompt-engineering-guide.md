---
title: Prompt Engineering — Nghệ thuật giao tiếp với LLM
description: >-
  Hướng dẫn toàn diện về Prompt Engineering: từ zero-shot, few-shot đến
  Chain-of-Thought và các kỹ thuật nâng cao để khai thác tối đa sức mạnh của
  LLM.
pubDate: '2026-05-23'
lang: vi
tags:
  - Prompt Engineering
  - LLM
  - AI
  - Tutorial
concepts:
  - Zero-shot Prompting
  - Few-shot Prompting
  - Chain-of-Thought
  - System Prompt
  - Temperature
  - Prompt Engineering
tools:
  - ChatGPT
  - Claude
  - Gemini
topics:
  - AI & Machine Learning
relations:
  - to: rag-explained
    type: SAME_TOPIC
  - to: llm-agents
    type: PREREQUISITE
readTime: 12
---
## Prompt Engineering là gì?

Prompt Engineering là kỹ thuật thiết kế và tối ưu hóa câu lệnh (prompt) đầu vào để LLM tạo ra output chính xác và hữu ích nhất có thể.

## Các kỹ thuật cơ bản

### 1. Zero-shot Prompting

Yêu cầu mô hình thực hiện tác vụ mà không cần ví dụ:

```
Dịch câu sau sang tiếng Anh: Trí tuệ nhân tạo đang thay đổi thế giới
```

### 2. Few-shot Prompting

Cung cấp một vài ví dụ để mô hình học pattern trước khi trả lời.

### 3. Chain-of-Thought (CoT)

Yêu cầu mô hình suy nghĩ từng bước một — cực kỳ hiệu quả với bài toán logic và toán học.

## System Prompt

Định nghĩa vai trò và ngữ cảnh cho mô hình ngay từ đầu hội thoại. System Prompt mạnh = output tốt hơn.

## Temperature

- **Thấp (0.1–0.3)**: Output xác định, phù hợp phân tích và coding
- **Cao (0.7–1.0)**: Output sáng tạo, phù hợp viết lách và brainstorming
