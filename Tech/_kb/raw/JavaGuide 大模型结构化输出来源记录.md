---
title: JavaGuide 大模型结构化输出来源记录
created: 2026-06-22
source_type: article
status: ingested
---

# JavaGuide 大模型结构化输出来源记录

## 来源

- 原文：[大模型结构化输出：从 JSON 契约到 Function Calling 落地](https://javaguide.cn/ai/llm-basis/structured-output-function-calling.html)
- 提供方式：用户于 2026-06-22 在对话中提供全文。
- 正式笔记：[[../../AI/Java 转 AI 应用开发/01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地]]

## 整理说明

- 将原文按“故障模式 → 契约层次 → 工具链路 → Schema 设计 → 失败处理 → 安全治理 → Java 落地”重构。
- 对 OpenAI、Anthropic、Gemini、MCP 与 JSON Schema 的时效性内容按 2026-06-22 官方文档复核。
- 修正了把可信运行时元数据交给模型生成的问题：用户、租户和幂等键改由认证上下文或服务端产生。
- Java 示例改为模型侧简化 Schema、服务端严格校验和脱敏审计的双层设计。

## 后续维护

- 目标框架确定后补 Spring AI 或 LangChain4j 的可运行示例。
- 模型供应商升级时复核 strict 行为、Schema 子集和调用关联字段。

