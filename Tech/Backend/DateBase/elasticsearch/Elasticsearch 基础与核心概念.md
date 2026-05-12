---
title: Elasticsearch 基础与核心概念
created: 2026-05-11
description: Elasticsearch 的定位、核心术语、倒排索引基础，以及它与关系型数据库的根本差异。
tags:
  - elasticsearch
  - fundamentals
  - inverted-index
  - mapping
  - backend
---
# Elasticsearch 基础与核心概念

这篇笔记整理 Elasticsearch 的最小认知闭环：它是什么、适合做什么、为什么它常被称为“非结构化数据库”、以及核心术语之间的关系。

官方/来源：

- Elasticsearch Guide：<https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html>
- raw source：[[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]

[TOC]

## 目录

- [Key Takeaways](#key-takeaways)
- [Elasticsearch 是什么](#elasticsearch-是什么)
- [为什么常被称为非结构化数据库](#为什么常被称为非结构化数据库)
- [全文检索基础](#全文检索基础)
- [核心概念](#核心概念)
- [Elasticsearch vs MySQL](#elasticsearch-vs-mysql)
- [适用场景与边界](#适用场景与边界)
- [最佳实践](#最佳实践)
- [Related Pages](#related-pages)
- [参考来源](#参考来源)

## Key Takeaways

- Elasticsearch 的本质是分布式搜索与分析引擎，不是事务型数据库替代品。
- 它对文本检索高效，依赖倒排索引、分词与相关性评分。
- 它常被称为“非结构化数据库”，更准确的说法是：面向半结构化与非结构化数据的弱 schema 搜索系统。
- 入门时最重要的概念不是 API，而是 `index / document / field / mapping / shard / replica`。

## Elasticsearch 是什么

Elasticsearch 是基于 Lucene 的分布式搜索和分析引擎，核心能力包括：

- 面向文档的 JSON 存储
- 全文检索
- 结构化过滤
- 聚合分析
- 近实时搜索
- 分布式扩展与高可用

在工程里，它最常见的定位有三类：

- 业务搜索引擎：商品、内容、知识库、日志检索
- 分析查询引擎：日志、指标、行为事件
- 读优化系统：承接查询、排序、推荐、检索类读流量

## 为什么常被称为非结构化数据库

“非结构化数据库”这个说法有直觉价值，但不够精确。更准确的理解是：

- 关系型数据库偏向 `Schema-on-write`
- Elasticsearch 偏向 `Schema-on-read`

两者区别不在于“有没有 schema”，而在于“schema 什么时候约束数据”。

### Schema-on-write

- 写入前先定义表结构
- 写入时强校验
- 类型不匹配直接失败

### Schema-on-read

- 文档先进入系统，再按 mapping 解释
- mapping 既可以显式定义，也可能由 dynamic mapping 推断
- 对半结构化和多变字段更友好，但错误更可能延后暴露

### 需要纠偏的认知

- Elasticsearch 不是完全无 schema
- mapping 也不是关系库里那种强门禁 schema
- 生产环境不应该放任 dynamic mapping 无边界扩张

## 全文检索基础

全文检索的核心流程可以概括为：

1. 文档进入索引
2. 文本被 analyzer 拆成 token
3. token 构造成倒排索引
4. 查询时先分析查询词，再到倒排索引里检索
5. 对候选文档做相关性计算和排序

### 正向索引 vs 倒排索引

| 结构 | 适合的问题 | 成本特征 |
| --- | --- | --- |
| 正向索引 | 根据 ID 找文档 | 写简单，按词搜索慢 |
| 倒排索引 | 根据词找文档 | 建索引复杂，但搜索快 |

倒排索引记录的是“词 -> 出现在哪些文档、哪些位置、频次如何”，因此全文检索能避免全表扫描。

## 核心概念

### 数据模型

- `Index`：一组文档的逻辑集合
- `Document`：最小存储单元，通常是 JSON
- `Field`：文档字段
- `Mapping`：定义字段类型、是否索引、如何分析、是否可聚合等

### 分布式模型

- `Node`：单个 Elasticsearch 实例
- `Cluster`：一组节点构成的集群
- `Shard`：索引的水平拆分单元
- `Replica`：主分片的副本，承担高可用和读扩展

### 检索模型

- `Analyzer`：文本分析器，由字符过滤器、分词器、token filter 组成
- `Query DSL`：JSON 风格查询语言
- `_score`：相关性分数
- `Aggregation`：统计与分组分析能力

## Elasticsearch vs MySQL

| Elasticsearch | MySQL | 更合适的语义 |
| --- | --- | --- |
| Index | Table | 逻辑集合 |
| Document | Row | 一条记录 |
| Field | Column | 字段 |
| Mapping | Schema | 结构与索引规则 |
| Query DSL | SQL | 查询表达方式 |

但类比不能过度：

- MySQL 解决事务、一致性、强约束
- Elasticsearch 解决搜索、排序、聚合、读扩展

如果把 ES 当作主交易库，会在事务、一致性、复杂关联、更新代价上踩坑。

## 适用场景与边界

### 典型适用场景

- 海量文本检索
- 日志与事件查询
- 商品搜索与筛选
- 多条件排序
- 聚合统计
- 地理位置检索

### 不擅长的事情

- 强事务写入
- 高频跨文档关联
- 复杂 join
- 强一致 OLTP

## 最佳实践

- 显式设计 mapping，不把 dynamic mapping 当作长期治理方案。
- 从一开始就区分 `text` 和 `keyword`。
- 把 ES 当作搜索与分析层，而不是关系数据库替代品。
- 对业务方讲清楚“近实时”和“事务提交成功”不是一回事。

## Related Pages

- [[Elasticsearch 阅读指南]]
- [[Elasticsearch 知识索引]]
- [[Elasticsearch 索引设计、文档与查询 DSL]]
- [[Elasticsearch 检索原理、NRT 与相关性评分]]
- [[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]

## 参考来源

- raw source：[[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]
- Elasticsearch Guide：<https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html>
