---
title: Prompt 工程
created: 2026-06-21
description: 将 Prompt 作为可版本化、可评测、可灰度的行为规格进行工程化管理。
tags: [ai, prompt-engineering, security, planned]
status: planned
---
# Prompt 工程

> 阅读状态：待精读。重点关注结构、版本、测试和 Prompt Injection 防御。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- Role、Task、Context、Format、Constraint、Example 各承担什么职责？
- System Prompt 与用户输入如何隔离？
- CoT、Few-Shot、ReAct 何时有收益，何时增加泄露和成本风险？

## 待整理大纲

1. Prompt 结构与模板
2. Few-Shot 与任务分解
3. 外置化、版本、灰度与回滚
4. Prompt Injection 与纵深防御
5. Prompt 回归评测

## 实践与验证

- [ ] 建立一个外置 Prompt 模板和版本号。
- [ ] 构造正常、边界与攻击输入集。
- [ ] 用 Golden Set 对比变更前后效果。

## 相关页面

- [[上下文工程]]
- [[../05-工程化/AI 应用评测体系]]
- [[../05-工程化/AI 应用安全与合规]]

## 参考来源

- <https://javaguide.cn/ai/agent/prompt-engineering.html>

