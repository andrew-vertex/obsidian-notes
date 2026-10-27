---
title: AI Agent 基础
created: 2026-06-21
description: Agent 的组成、Tool Loop、ReAct、Plan-and-Execute、Reflection 与生产边界。
tags: [ai, agent, tool-calling, planned]
status: planned
---
# AI Agent 基础

> 阅读状态：待精读。Agent 的关键不是“能调工具”，而是动态决策过程是否有界、可恢复、可审计。


## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- Agent 与固定 Workflow 的边界是什么？
- ReAct、Plan-and-Execute、Reflection 各适合什么任务？
- 循环终止、状态持久化和 Human-in-the-Loop 如何设计？

## 待整理大纲

1. Agent 核心组件与运行循环
2. 工具选择、参数和观察结果
3. 常见 Agent 范式
4. 状态、检查点与恢复
5. 有界自治和人工审批

## 实践与验证

- [ ] 实现带最大步数和总超时的最小 Agent。
- [ ] 注入工具超时、非法参数和重复执行。

## 相关页面

- [[../01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地]]
- [[Workflow、Graph 与 Loop]]
- [[Harness Engineering]]

## 参考来源

- <https://javaguide.cn/ai/agent/agent-basis.html>
