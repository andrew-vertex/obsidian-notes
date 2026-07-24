---
title: Elasticsearch 知识索引
created: 2026-05-11
description: Elasticsearch 主题的总入口，覆盖基础概念、Mapping、查询、分词、原理、分页、集群与数据同步。
tags:
  - elasticsearch
  - index
  - search
  - backend
  - database
---
# Elasticsearch 知识索引

这组笔记把一批 Notion 导出的 Elasticsearch 历史笔记重新编译成 Tech 域下的正式知识页。它既保留原有覆盖面，也对过时做法做了版本纠偏，使内容更适合作为长期参考。

如果你不确定阅读顺序，先看 [[Elasticsearch 阅读指南]]。

官方/来源：

- Elasticsearch Reference：<https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html>
- Java API Client：<https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html>
- 域内 raw 记录：[[.raw/articles/Elasticsearch Notion 导出来源记录]]


## 目录

- [Key Takeaways](#key-takeaways)
- [推荐阅读路径](#推荐阅读路径)
- [知识地图](#知识地图)
- [版本与最佳实践说明](#版本与最佳实践说明)
- [Comparison Table](#comparison-table)
- [Related Pages](#related-pages)
- [参考来源](#参考来源)

## Key Takeaways

- 这套笔记以 `Elasticsearch` 作为“搜索与分析引擎”来组织，而不是把它误当作事务型数据库替代品。
- 原始来源以 `7.17.3` 为主，但正式知识页会对客户端、分页、集群角色与安全配置按当前官方最佳实践做标注。
- 建议把知识分成四层理解：数据模型、检索与分析、分布式运行机制、工程演进与同步。
- canonical page 放在 `Tech/Backend/DateBase/elasticsearch/`，raw 证据保留在 `.raw/data/elasticsearch-notion-export/`。

## 推荐阅读路径

### 新手路径

1. [[Elasticsearch 基础与核心概念]]
2. [[Elasticsearch 索引设计、文档与查询 DSL]]
3. [[Elasticsearch 分词、中文检索与自动提示]]
4. [[Elasticsearch 检索原理、NRT 与相关性评分]]

### 架构与底层路径

1. [[Elasticsearch 基础与核心概念]]
2. [[Elasticsearch 检索原理、NRT 与相关性评分]]
3. [[Elasticsearch 深分页与结果遍历]]
4. [[Elasticsearch 集群与分布式架构]]

### 工程落地路径

1. [[Elasticsearch 索引设计、文档与查询 DSL]]
2. [[Elasticsearch 分词、中文检索与自动提示]]
3. [[Elasticsearch 数据同步、别名与重建索引]]
4. [[Elasticsearch 集群与分布式架构]]

## 知识地图

- [[Elasticsearch 基础与核心概念]]：是什么、适合做什么、核心概念、为什么它常被归为半结构化搜索系统。
- [[Elasticsearch 索引设计、文档与查询 DSL]]：索引、Mapping、文档 CRUD、Query DSL、聚合、Java 客户端与操作面。
- [[Elasticsearch 分词、中文检索与自动提示]]：分词器、IK、自定义词典、拼音分析、completion / term / phrase suggest。
- [[Elasticsearch 检索原理、NRT 与相关性评分]]：倒排索引、写入链路、NRT、translog、segment merge、BM25、评分控制。
- [[Elasticsearch 深分页与结果遍历]]：`from/size`、`search_after`、PIT、scroll、深分页选型。
- [[Elasticsearch 集群与分布式架构]]：节点角色、分片、副本、路由、故障转移、监控与健康治理。
- [[Elasticsearch 数据同步、别名与重建索引]]：alias、reindex、零停机索引迁移、MySQL 到 ES 同步架构、Canal 与 MQ。

## 版本与最佳实践说明

- 原始来源以 `7.17.3` 为主，因此里面会看到 `RestHighLevelClient`、`node.master` / `node.data` 等历史写法。
- 当前官方实践里：
  - Java 侧优先使用 `Java API Client`，`RestHighLevelClient` 仅保留给遗留 7.x 项目。
  - 深分页的交互式查询优先 `search_after + PIT`，而不是 scroll。
  - 新建集群优先使用 `node.roles` 管理节点角色，而不是旧式布尔角色开关。
  - 8.x 中安全能力默认开启，部署文档里不应再把“裸开放 9200”当成默认方案。

## Comparison Table

| 关注面 | 推荐页面 | 典型问题 |
| --- | --- | --- |
| 概念入门 | [[Elasticsearch 基础与核心概念]] | ES 到底是什么，和 MySQL 差在哪 |
| 建模与查询 | [[Elasticsearch 索引设计、文档与查询 DSL]] | text/keyword 怎么区分，DSL 怎么写 |
| 中文搜索 | [[Elasticsearch 分词、中文检索与自动提示]] | IK、拼音、自动补全怎么做 |
| 原理与排序 | [[Elasticsearch 检索原理、NRT 与相关性评分]] | 为什么能快，为什么这么排 |
| 深分页 | [[Elasticsearch 深分页与结果遍历]] | `from/size` 为什么慢，什么时候用 PIT |
| 集群与可用性 | [[Elasticsearch 集群与分布式架构]] | 分片、副本、故障转移、节点职责 |
| 数据演进 | [[Elasticsearch 数据同步、别名与重建索引]] | reindex、alias、Canal、同步链路 |

## Related Pages

- [[Elasticsearch 阅读指南]]
- [[wiki/learning/_index]]
- [[.raw/articles/Elasticsearch Notion 导出来源记录]]

## 参考来源

- Elasticsearch Reference：<https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html>
- Java API Client：<https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html>
- raw source record：[[.raw/articles/Elasticsearch Notion 导出来源记录]]
