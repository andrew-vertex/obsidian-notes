---
title: LLM API 调用工程
created: 2026-06-21
description: Java 服务端接入 LLM 时的流式响应、超时重试、异常分层与多模型适配。
tags: [ai, llm, api, java, planned]
status: planned
---
# LLM API 调用工程

> 阅读状态：待精读。目标是把模型调用封装为可替换、可观测、可降级的基础设施组件。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- SSE、WebFlux 与同步调用各适合什么场景？
- 超时、429、供应商 5xx、内容拒绝和解析失败如何分类？
- 如何隔离供应商 SDK，并实现 fallback 与成本归因？

## 待整理大纲

1. 请求与响应协议
2. 流式传输和代理配置
3. 超时、重试、熔断与限流
4. 多模型适配与领域接口
5. 调用日志、Token 与 Trace

## 实践与验证

- [ ] 完成非流式和 SSE 调用。
- [ ] 对超时、429、500、网络中断做故障注入。
- [ ] 验证切换模型只影响实现层。

## 相关页面

- [[Java AI 框架选型]]
- [[大模型结构化输出：从 JSON 契约到 Function Calling 落地]]
- [[../05-工程化/大模型网关]]

## 参考来源

- <https://javaguide.cn/ai/llm-basis/llm-api-engineering.html>
