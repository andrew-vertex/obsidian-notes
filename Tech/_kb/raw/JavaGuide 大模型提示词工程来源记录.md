---
title: JavaGuide 大模型提示词工程来源记录
created: 2026-06-22
source_type: article
status: ingested
---

# JavaGuide 大模型提示词工程来源记录

## 来源

- 原文：[大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？](https://javaguide.cn/ai/agent/prompt-engineering.html)
- 提供方式：用户于 2026-06-22 在对话中提供全文。
- 正式笔记：[[../../AI/Java 转 AI 应用开发/02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？]]

## 整理说明

- 将内容按“基本结构 → 提示技巧 → 长文本与幻觉 → 工程生命周期 → 安全 → Agent 上下文”重构。
- 把 CoT 调整为可验证依据和检查点，避免把完整内部推理当作生产接口。
- 将 Prompt 从写作技巧提升为具备版本、Golden Set、灰度、回滚和观测的工程资产。
- 对 Spring AI、OpenAI、Anthropic 与 Lost in the Middle 的时效性内容按 2026-06-22 官方文档或论文复核。
- Spring AI 原文的 1.1.x 版本范围未固化到正式页；示例改按当前 2.0.0 参考文档标注，并保留版本复核提醒。

## 后续维护

- 目标项目确定 Spring AI BOM 后补可编译示例。
- 模型或 SDK 升级时复核 reasoning、预填充、结构化输出和消息角色行为。

