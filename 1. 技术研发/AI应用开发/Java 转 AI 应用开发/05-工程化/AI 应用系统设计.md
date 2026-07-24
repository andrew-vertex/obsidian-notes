---
title: AI 应用系统设计
created: 2026-06-21
description: 将模型、Prompt、RAG、Memory、Tools、Gateway、Evaluation 与治理组合为生产架构。
tags: [ai, system-design, architecture, planned]
status: planned
---
# AI 应用系统设计

> 阅读状态：待精读。目标是从 Prompt Demo 过渡到有 SLA、成本和安全边界的服务。


## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- 模型接入、上下文、RAG、Memory 和 Tool 层如何划分？
- 哪些能力应集中到 Gateway，哪些留在业务服务？
- 在线链路、离线知识管道和评测反馈如何闭环？

## 待整理大纲

1. 需求与质量属性
2. 在线请求架构
3. 离线知识与评测管道
4. 稳定性、观测和成本
5. 安全、合规与多租户

## 实践与验证

- [ ] 为项目绘制组件图、序列图和故障边界。
- [ ] 记录至少三个关键 ADR。

## 相关页面

- [[大模型网关]]
- [[AI 应用评测体系]]
- [[AI 应用安全与合规]]

## 参考来源

- <https://javaguide.cn/ai/system-design/ai-application-architecture.html>

