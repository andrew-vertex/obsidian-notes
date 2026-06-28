---
title: AI 应用安全与合规
created: 2026-06-21
description: AI 应用的 Prompt Injection、防越权、工具审批、PII 脱敏、审计和数据留存。
tags: [ai, security, compliance, guardrails, planned]
status: planned
---
# AI 应用安全与合规

> 阅读状态：待专题补充。本页根据路线文章先建立骨架，后续应补充权威安全与合规来源。

[TOC]

## Key Takeaways

- 待补充来源并完成威胁建模后填写。

## 阅读问题

- 直接/间接 Prompt Injection 能影响哪些资产和执行路径？
- 模型认知层约束、代码执行层权限和人工决策层如何纵深防御？
- PII、日志、Trace、RAG ACL 和长期记忆的留存/删除如何治理？

## 待整理大纲

1. 资产、信任边界与威胁模型
2. 输入、上下文和输出安全
3. 工具权限、沙箱与人工审批
4. RAG/Memory 的租户和 ACL 隔离
5. 脱敏、审计、留存和合规

## 实践与验证

- [ ] 为项目完成一次威胁建模。
- [ ] 构造越狱、间接注入、越权检索和危险工具测试。
- [ ] 验证 PII 脱敏与用户删除链路。

## 相关页面

- [[../02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？]]
- [[../04-Agent/AI Agent 基础]]
- [[AI 应用系统设计]]

## 参考来源

- [[.raw/articles/JavaGuide Java Go 开发者 AI 应用开发路线来源记录]]
- 权威安全标准与适用法规：`需确认`。
