---
title: RAG 检索优化
created: 2026-06-21
description: Query Rewrite、混合检索、RRF、Rerank 与上下文压缩的组合和评测。
tags: [ai, rag, retrieval, rerank, planned]
status: planned
---
# RAG 检索优化

> 阅读状态：待精读。每次优化必须绑定基准集，避免凭主观样例判断。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- Query Rewrite、Multi-Query、HyDE 分别改善哪类问题？
- 向量检索、BM25、RRF 和 Rerank 如何分工？
- `recall_top_k`、`rerank_top_n`、`context_top_n` 如何基于指标确定？

## 待整理大纲

1. 查询理解与改写
2. 稀疏、稠密与混合检索
3. RRF 与 Rerank
4. 上下文压缩和去重
5. 检索指标、生成指标与错误分析

## 实践与验证

- [ ] 对比纯向量、BM25、Hybrid + Rerank。
- [ ] 记录 Hit Rate@K、MRR、Context Precision/Recall。

## 相关页面

- [[RAG 基础]]
- [[../05-工程化/AI 应用评测体系]]
- [[../02-Prompt 与上下文/上下文工程]]

## 参考来源

- <https://javaguide.cn/ai/rag/rag-optimization.html>

