---
title: RAG 向量索引与向量数据库
created: 2026-06-21
description: Embedding 存储、ANN 索引算法与 pgvector、Milvus、Elasticsearch 等选型维度。
tags: [ai, rag, vector-database, embedding, planned]
status: planned
---
# RAG 向量索引与向量数据库

> 阅读状态：待精读。选型结论需结合数据规模、过滤、延迟、运维和现有技术栈。

[TOC]

## Key Takeaways

- 待读完来源并实测后填写。

## 阅读问题

- Cosine、Dot Product、L2 分别适用于什么归一化条件？
- HNSW、IVF/IVFFLAT 的精度、延迟、内存和构建成本如何权衡？
- pgvector、Milvus、Elasticsearch 在混合检索和 Metadata 过滤上有何差异？

## 待整理大纲

1. 向量表示和距离度量
2. 精确检索与 ANN
3. HNSW 与 IVF 系列
4. 数据库选型矩阵
5. 容量、索引参数和基准测试

## 实践与验证

- [ ] 估算向量、索引和 Metadata 容量。
- [ ] 在真实语料上测 Recall、P95 延迟和写入成本。

## 相关页面

- [[RAG 基础]]
- [[RAG 检索优化]]
- [[RAG 知识库更新]]

## 参考来源

- <https://javaguide.cn/ai/rag/rag-vector-store.html>

