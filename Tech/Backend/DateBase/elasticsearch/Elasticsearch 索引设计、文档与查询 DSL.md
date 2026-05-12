---
title: Elasticsearch 索引设计、文档与查询 DSL
created: 2026-05-11
description: Elasticsearch 的索引设计、字段建模、文档操作、查询 DSL、聚合与 Java 客户端选型。
tags:
  - elasticsearch
  - mapping
  - query-dsl
  - aggregation
  - java
---
# Elasticsearch 索引设计、文档与查询 DSL

这篇笔记整理 Elasticsearch 的日常工程面：索引怎么建、字段怎么选、文档如何操作、DSL 怎么写、聚合怎么做，以及 Java 客户端在新旧项目里的选型边界。

官方/来源：

- Mapping：<https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html>
- Query DSL：<https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html>
- Java API Client：<https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html>
- raw source：[[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]

[TOC]

## 目录

- [Key Takeaways](#key-takeaways)
- [索引与 Mapping 设计](#索引与-mapping-设计)
- [文档操作](#文档操作)
- [Query DSL 基础](#query-dsl-基础)
- [聚合分析](#聚合分析)
- [Java 客户端选型](#java-客户端选型)
- [最佳实践](#最佳实践)
- [Related Pages](#related-pages)
- [参考来源](#参考来源)

## Key Takeaways

- Elasticsearch 的建模核心不是“建表”，而是“让字段适配检索、过滤、排序和聚合”。
- `text` 与 `keyword` 的区分，是绝大多数 Mapping 设计的起点。
- Mapping 一旦落地，字段类型基本不可原地修改；设计失误通常要靠 `reindex + alias` 修复。
- 查询时优先把“必须过滤的条件”放进 `filter`，把需要参与相关性计算的条件放进 `must` / `should`。

## 索引与 Mapping 设计

### 常见字段类型

| 类型 | 适合用途 | 说明 |
| --- | --- | --- |
| `text` | 全文检索 | 会被分析，不适合直接排序和聚合 |
| `keyword` | 精确匹配、排序、聚合 | 不分词 |
| `integer` / `long` / `double` | 数值查询 | 适合范围过滤与统计 |
| `date` | 时间字段 | 适合范围过滤与时间直方图 |
| `boolean` | 状态字段 | 适合过滤 |
| `geo_point` | 地理位置 | 支持距离与地理边界查询 |
| `nested` | 嵌套对象数组 | 适合需要保留对象边界的场景 |

### 最常见的 text / keyword 设计

```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

解释：

- `title` 用于全文搜索
- `title.keyword` 用于排序、聚合和精确过滤

### 常见 Mapping 参数

- `analyzer`：建索引时如何分词
- `search_analyzer`：查询时如何分词
- `index`：是否建立倒排索引
- `norms`：是否保留长度归一化信息
- `copy_to`：把多个字段复制到综合检索字段
- `fields`：同字段多种索引方式
- `eager_global_ordinals`：高频 terms 聚合时可考虑开启

### dynamic mapping 的边界

原始笔记保留了 dynamic mapping 的便利性，但生产环境要有边界：

- 字段多变但可控：`dynamic: true`
- 明确不接受未知字段：`dynamic: strict`
- 特别关注“字段爆炸”和跨文档字段类型冲突

### Mapping 变更的现实

- 可新增字段
- 不适合直接修改已有字段类型
- 大多数“类型改错了”的场景，正确姿势是创建新索引后 `reindex`

### 历史与版本说明

- 原始来源里仍然会出现 `_doc`、type 历史背景等内容。
- 在现代 Elasticsearch 中，mapping types 已被移除，新建索引不要再以“一个 index 多个 type”为模型思考。

## 文档操作

### 索引操作

- 创建索引：`PUT /index-name`
- 查看索引：`GET /index-name`
- 查看 mapping：`GET /index-name/_mapping`
- 修改动态 settings：`PUT /index-name/_settings`
- 删除索引：`DELETE /index-name`

### 文档 CRUD

- 创建或覆盖：`PUT /index/_doc/{id}`
- 读取：`GET /index/_doc/{id}`
- 局部更新：`POST /index/_update/{id}`
- 删除：`DELETE /index/_doc/{id}`
- 批量写入：`POST /_bulk`

### Bulk 使用注意

- 请求体必须是 NDJSON
- 单批不宜过大，要按体积与耗时拆分
- `refresh=true` 只该用于确实需要马上可见的少量写入
- 导入与回填通常配合批量、限速、监控、失败重试

## Query DSL 基础

### URI Query vs Request Body

- `URI Query` 适合临时测试
- `Request Body` 才适合正式业务查询

正式项目中应优先使用 JSON Query DSL，因为它更清晰、更易组合、更便于代码生成。

### 常见查询类型

| 查询 | 适合用途 | 说明 |
| --- | --- | --- |
| `term` / `terms` | 精确过滤 | 针对 `keyword`、数值、日期、布尔 |
| `match` | 单字段全文检索 | 会分词并算分 |
| `multi_match` | 多字段全文检索 | 多字段匹配 |
| `range` | 范围过滤 | 时间、价格、分值 |
| `match_phrase` | 短语检索 | 可配 `slop` |
| `bool` | 组合查询 | `must` / `should` / `filter` / `must_not` |
| `regexp` | 正则匹配 | 慎用，代价高 |
| `geo_distance` | 地理半径检索 | 位置搜索 |
| `geo_bounding_box` | 地理框选 | 地图区域筛选 |

### bool 查询的工程理解

- `must`：必须匹配，参与评分
- `should`：可选匹配，常用于加权
- `filter`：必须匹配，但不参与评分，且更利于缓存
- `must_not`：排除条件

一个实用原则：

- 只要条件本质是“过滤”，优先放 `filter`
- 只要条件本质是“提升相关性”，才放 `must` / `should`

### 排序、高亮、分页

- 不指定 `sort` 时，默认按 `_score`
- 指定字段排序后，结果往往不再依赖相关性评分
- 高亮适合面向用户展示，不适合作为业务判断依据
- `from / size` 适合浅分页；深分页单独见 [[Elasticsearch 深分页与结果遍历]]

### Query Profile

当复杂查询性能不达预期时，可以打开 `profile` 分析：

- 各子句耗时
- 收集阶段开销
- 评分阶段开销
- 哪一段 query 真正慢

这比靠感觉优化更可靠。

## 聚合分析

### 三类聚合

- `Bucket`：分桶分组，例如 `terms`、`range`、`date_histogram`
- `Metric`：统计值，例如 `sum`、`avg`、`min`、`max`
- `Pipeline`：基于其他聚合结果继续计算，例如累计和、导数

### 典型组合

- `terms + avg`：按品牌分组后求均价
- `date_histogram + sum`：按月汇总销售额
- `terms + cardinality`：按分类分组后统计唯一用户数

### 聚合设计注意点

- 不要直接对 `text` 字段聚合
- 高频 `terms` 聚合可以考虑 `eager_global_ordinals`
- 大字段聚合要关注内存与 segment 成本

## Java 客户端选型

原始来源里记录了 `RestHighLevelClient` 的大量代码示例，这对理解请求组装仍有价值，但正式工程建议要区分版本：

### 新项目

- 优先使用官方 `Java API Client`
- 好处是版本演进更明确、类型更完整、与当前文档同步

### 遗留 7.x 项目

- `RestHighLevelClient` 仍可能存在于历史代码中
- 适合做兼容维护，但不建议作为新系统默认选择

### 客户端之外真正重要的事

- 让查询语义先在 Kibana / Console 跑通
- 把 mapping、query、aggregation 当成“检索协议”而不是“Java API 问题”

## 最佳实践

- 先设计查询模式，再设计 Mapping，而不是反过来。
- `copy_to` 常常比盲目 `multi_match` 更稳定。
- `keyword` 字段负责精确过滤、排序、聚合，`text` 字段负责全文检索。
- 需要版本演进的索引，预先规划 alias，避免未来切换成本过高。

## Related Pages

- [[Elasticsearch 阅读指南]]
- [[Elasticsearch 知识索引]]
- [[Elasticsearch 基础与核心概念]]
- [[Elasticsearch 分词、中文检索与自动提示]]
- [[Elasticsearch 数据同步、别名与重建索引]]
- [[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]

## 参考来源

- Mapping：<https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html>
- Query DSL：<https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html>
- Java API Client：<https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html>
- raw source：[[../../../_kb/raw/Elasticsearch Notion 导出来源记录]]
