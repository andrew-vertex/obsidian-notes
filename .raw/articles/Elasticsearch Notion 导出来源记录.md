---
title: Elasticsearch Notion 导出来源记录
created: 2026-05-11
description: 记录从 Temp 提升到 Tech 域 raw 层的 Elasticsearch Notion 导出来源与编译边界。
tags:
  - source-record
  - elasticsearch
  - notion-export
  - raw
---
# Elasticsearch Notion 导出来源记录

这份 source record 对应一组从 `Temp/` 提升到 `Tech/_kb/raw/` 的 Elasticsearch 历史笔记导出。原始导出保持不改写，正式知识页在 `Tech/Backend/DateBase/elasticsearch/` 下维护。

[TOC]

## Source Inventory

- 根目录页：`Tech/_kb/raw/elasticsearch-notion-export/ElasticSearch 7eb4c053a36543cd89abd68f422009c9.md`
- 基础总页：`Tech/_kb/raw/elasticsearch-notion-export/ElasticSearch/ElasticSearch 基础 e070034f69134baa8468905db0db3895.md`
- 使用总页：`Tech/_kb/raw/elasticsearch-notion-export/ElasticSearch/ElasticSearch 使用 6eeae56966cf41be9baa42b9dfc765c3.md`
- 原理总页：`Tech/_kb/raw/elasticsearch-notion-export/ElasticSearch/ElasticSearch 原理 08864454f1f943d283a8599774c60dcf.md`
- 高级总页：`Tech/_kb/raw/elasticsearch-notion-export/ElasticSearch/ElasticSearch 高级 2ec7ebc4e86e432f8e086b634fcbdc94.md`
- 子页面：
  - `Elasticsearch 为何被称为“非结构化数据库”`
  - `Elasticsearch 评分机制与权重控制笔记`
  - `ElasticSearch 深度分页问题详情`
  - `Search After 原理`
  - `Scroll 原理分析`
- 截图：原始导出目录中的 `.png` 文件全部保留，作为当时的 Kibana、集群与查询示意图证据层。

## Provenance Notes

- 原始内容主要围绕 `7.17.3` 版本展开，包含 Docker 部署、`RestHighLevelClient`、IK 分词器、pinyin 插件、scroll / search_after、分片、副本、Canal 同步等主题。
- 原始示例中存在历史环境的 IP、用户名与示例密码。这些内容保留在 raw 层，但正式知识页统一抽象为占位值与最佳实践说明，不把历史环境参数当作推荐做法。
- 原始内容中也保留了 7.x 时代的若干写法，例如角色布尔配置、`RestHighLevelClient`、scroll 作为深分页方案等；正式知识页会保留其历史背景，同时按当前官方文档更新最佳实践。

## Compilation Targets

- [[../../Backend/DateBase/elasticsearch/Elasticsearch 阅读指南]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 知识索引]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 基础与核心概念]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 索引设计、文档与查询 DSL]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 分词、中文检索与自动提示]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 检索原理、NRT 与相关性评分]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 深分页与结果遍历]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 集群与分布式架构]]
- [[../../Backend/DateBase/elasticsearch/Elasticsearch 数据同步、别名与重建索引]]

## Curation Rules

- raw 层只保留证据，不在这里做重写和观点纠偏。
- 正式页优先合并重复知识，避免把 Notion 的目录页原样平铺。
- 同一知识点只保留一个 canonical page，再通过索引和 Related Pages 做路由。

## 参考来源

- vault raw：`Tech/_kb/raw/elasticsearch-notion-export/`
- compiled wiki：[[../../Backend/DateBase/elasticsearch/Elasticsearch 阅读指南]]
