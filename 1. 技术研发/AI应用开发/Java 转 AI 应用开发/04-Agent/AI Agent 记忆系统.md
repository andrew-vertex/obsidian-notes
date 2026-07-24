---
title: AI Agent 记忆系统
created: 2026-06-21
description: Agent 短期记忆、长期记忆、摘要压缩、检索注入、隔离与遗忘机制。
tags: [ai, agent, memory, context-engineering, planned]
status: planned
---
# AI Agent 记忆系统

> 阅读状态：待精读。记忆不是无限追加聊天记录，而是有选择地写入、检索、更正和遗忘。


## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- 短期状态、对话历史、长期事实和 RAG 知识如何区分？
- 哪些内容值得写入长期记忆，如何避免错误固化？
- 多租户隔离、衰减、删除和用户更正如何实现？

## 待整理大纲

1. 记忆类型与职责
2. 写入提取、置信度与幂等
3. 检索、排序与上下文注入
4. 摘要压缩、衰减和遗忘
5. 隐私、租户隔离与可更正性

## 实践与验证

- [ ] 对比滑动窗口、摘要和长期语义记忆。
- [ ] 验证跨租户不可检索、用户删除可生效。

## 相关页面

- [[../02-Prompt 与上下文/上下文工程]]
- [[AI Agent 基础]]
- [[../03-RAG/RAG 基础]]

## 参考来源

- <https://javaguide.cn/ai/agent/agent-memory.html>

