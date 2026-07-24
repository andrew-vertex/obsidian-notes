---
title: 多工具 Agent 项目
created: 2026-06-21
description: 用多工具、记忆、检查点、审批和错误恢复验证生产级 Agent 的核心能力。
tags: [ai, agent, project, java, planned]
status: planned
---
# 多工具 Agent 项目

> 项目状态：待启动。工具默认只读；写操作必须增加明确审批和幂等控制。

[TOC]

## 项目目标

- Agent 能调用数据库查询、知识库检索和 Web 搜索等至少 3 个工具。
- 任务中断后能从检查点恢复。
- 对参数错误、工具超时和模型失败有有限重试与降级。

## 最小范围

- 有界 ReAct 或 Plan-and-Execute Loop。
- 会话状态、短期记忆、工具 Trace 和成本预算。
- 高风险工具审批、最大步数、总超时和终止原因。

## 验收指标

| 维度 | 指标 | 目标值 |
| --- | --- | --- |
| 工具 | 选择/参数准确率 | 待基线测试后填写 |
| 稳定性 | 错误恢复率 | 待基线测试后填写 |
| 效率 | 平均步骤/Token/成本 | 待基线测试后填写 |
| 安全 | 未授权执行次数 | 0 |

## 待办

- [ ] 定义工具 Schema、权限和幂等语义。
- [ ] 设计 Agent State 与 Checkpoint。
- [ ] 建立轨迹级评测集。
- [ ] 注入循环、超时、重复调用和越权故障。

## 相关页面

- [[../04-Agent/AI Agent 基础]]
- [[../04-Agent/Workflow、Graph 与 Loop]]
- [[../05-工程化/AI 应用安全与合规]]

## 参考来源

- [[.raw/articles/JavaGuide Java Go 开发者 AI 应用开发路线来源记录]]
