---
title: RAG 文档处理与切分
created: 2026-06-21
description: RAG 离线管道中的解析、清洗、结构恢复、切分与 Metadata 设计。
tags: [ai, rag, document-processing, chunking, planned]
status: planned
---
# RAG 文档处理与切分

> 阅读状态：待精读。重点验证文档进库前是否已经丢失结构或语义。

[TOC]

## Key Takeaways

- 待读完来源后填写。

## 阅读问题

- PDF、Office、扫描件和表格分别需要什么解析策略？
- 固定长度、递归、标题层级和语义切分如何选择？
- Chunk 的 Metadata 如何支持权限、版本、过滤与引用？

## 待整理大纲

1. 文档类型与解析器
2. 清洗和结构恢复
3. 切分策略与 Overlap
4. Metadata 与权限预过滤
5. 离线管道、幂等和质量检查

## 实践与验证

- [ ] 对同一文档比较两种切分策略。
- [ ] 人工抽检标题、表格、页码和引用是否保留。

## 相关页面

- [[RAG 基础]]
- [[RAG 知识库更新]]
- [[RAG 检索优化]]

## 参考来源

- <https://javaguide.cn/ai/rag/rag-document-processing.html>

