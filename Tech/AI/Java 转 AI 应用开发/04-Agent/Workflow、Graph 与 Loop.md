---
title: Workflow、Graph 与 Loop
created: 2026-06-21
description: AI 工作流中确定性 Workflow、状态 Graph 与有界 Agent Loop 的组合方式。
tags: [ai, agent, workflow, graph, planned]
status: planned
---
# Workflow、Graph 与 Loop

> 阅读状态：待精读。目标是把确定性主流程与不确定性 Agent 子循环分层治理。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- Workflow、Graph、Loop 分别负责什么控制问题？
- Node、Edge、State、Checkpoint 和 Replan 如何建模？
- 最大轮次、超时、预算和中断恢复应放在哪一层？

## 待整理大纲

1. 三种控制结构的边界
2. Graph 状态和节点契约
3. Agent Loop 的进入与退出
4. 检查点、恢复和人工审批
5. Spring AI Alibaba Graph 等 Java 方案

## 实践与验证

- [ ] 构建一个确定性主流程 + 单个 Agent 子循环。
- [ ] 服务重启后从检查点继续执行。

## 相关页面

- [[AI Agent 基础]]
- [[Harness Engineering]]
- [[../05-工程化/AI 应用系统设计]]

## 参考来源

- <https://javaguide.cn/ai/agent/workflow-graph-loop.html>

