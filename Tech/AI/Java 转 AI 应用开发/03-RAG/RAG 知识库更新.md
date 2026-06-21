---
title: RAG 知识库更新
created: 2026-06-21
description: RAG 知识库的增量同步、版本管理、删除、重建与一致性策略。
tags: [ai, rag, knowledge-base, data-pipeline, planned]
status: planned
---
# RAG 知识库更新

> 阅读状态：待精读。目标是让知识库更新可追踪、可回滚、可重建。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- 新增、修改、删除文档如何映射到 Chunk 与向量？
- Embedding 模型升级时如何双写、重建和切换？
- 在线检索如何避免看到混合版本或半成品索引？

## 待整理大纲

1. 文档身份与内容哈希
2. 增量更新和删除传播
3. Embedding/Chunk 策略版本
4. 全量重建与蓝绿切换
5. 一致性、审计和失败恢复

## 实践与验证

- [ ] 实现文档增删改和幂等重放。
- [ ] 演练一次 Embedding 模型版本迁移。

## 相关页面

- [[RAG 文档处理与切分]]
- [[RAG 向量索引与向量数据库]]
- [[../05-工程化/AI 应用系统设计]]

## 参考来源

- <https://javaguide.cn/ai/rag/rag-knowledge-update.html>

