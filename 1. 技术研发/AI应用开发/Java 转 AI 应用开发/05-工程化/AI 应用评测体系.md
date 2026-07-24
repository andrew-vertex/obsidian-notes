---
title: AI 应用评测体系
created: 2026-06-21
description: Prompt、RAG 与 Agent 的离线评测、线上实验、Trace 回放和回归门禁。
tags: [ai, evaluation, rag, agent, planned]
status: planned
---
# AI 应用评测体系

> 阅读状态：待精读。评测是各阶段的横切能力，不应等到上线前再补。


## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- Golden Set 如何从真实流量、边缘样本和失败案例构建？
- 检索、生成、工具调用和端到端成功分别用什么指标？
- LLM-as-a-Judge 的偏差如何校准，线上 A/B 如何控制风险？

## 待整理大纲

1. 评测目标与数据集
2. Prompt/结构化输出评测
3. RAG 检索与生成评测
4. Agent 轨迹与工具评测
5. CI 回归、线上灰度和 Trace 回放

## 实践与验证

- [ ] 建立 50～200 条起步 Golden Set。
- [ ] 将评测结果绑定 Prompt、模型和知识库版本。
- [ ] 人工抽检自动评审器的一致性。

## 相关页面

- [[../03-RAG/RAG 检索优化]]
- [[../04-Agent/AI Agent 基础]]
- [[AI 应用系统设计]]

## 参考来源

- <https://javaguide.cn/ai/llm-basis/llm-evaluation.html>

