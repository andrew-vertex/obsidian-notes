---
title: Personal Knowledge Base Operating Guide
created: 2026-05-07
description: 用于持续搭建、整理、更新个人知识库的中文操作手册。
tags:
  - schema
  - operating-guide
  - knowledge-base
  - obsidian
---
# Personal Knowledge Base Operating Guide

这份手册只回答一个问题：在当前 `/Notes` vault 里，后续应该如何稳定地接收资料、提升原始来源、整理正式笔记，并持续维护知识库。

## 目录

- [Key Takeaways](#key-takeaways)
- [一张图理解当前结构](#一张图理解当前结构)
- [四层模型](#四层模型)
- [标准流程：从 clipping 到正式知识页](#标准流程从-clipping-到正式知识页)
- [如何选择目录](#如何选择目录)
- [什么时候写 raw](#什么时候写-raw)
- [什么时候改 schema](#什么时候改-schema)
- [什么时候更新 index 和 log](#什么时候更新-index-和-log)
- [推荐日常工作流](#推荐日常工作流)
- [推荐提示词模板](#推荐提示词模板)
- [常见误区](#常见误区)
- [参考来源](#参考来源)

## Key Takeaways

- `Clippings/` 是 inbox，不是 canonical raw 层。
- root `_kb/` 是全局控制面，不是整个 vault 的正式笔记区。
- `Tech/_kb/`、`Tools/_kb/` 是领域控制面；`Tech/`、`Tools/` 现有内容目录才是正式 wiki 层。
- raw 不是简单复制一份全文，而是“可追溯来源层”。
- schema 不需要每篇笔记都改，只有结构规则真的变化时才更新。

## 一张图理解当前结构

```text
Notes/
  Clippings/                # 捕获入口 / inbox
  _kb/                      # vault 级控制面
    raw/
    schema/
    wiki/
    log.md
  Tech/
    _kb/                    # Tech 领域控制面
      raw/
      schema/
      index.md
      log.md
    Agent/                  # 正式笔记区
    Architecture/
    Backend/
  Tools/
    _kb/
      raw/
      schema/
      index.md
      log.md
    AI/
    Mac/
```

## 四层模型

### 1. Inbox 层

- 目录：`Clippings/`
- 用途：先收资料，不做最终分类判断

### 2. Raw 层

- 目录：`_kb/raw/` 或某个领域的 `_kb/raw/`
- 用途：保存“被提升后的来源记录”与高价值原始资料

### 3. Wiki 层

- 目录：`Tech/Agent/`、`Tools/AI/` 等现有正式笔记目录
- 用途：放最终可阅读、可链接、可维护的知识页

### 4. Schema 层

- 目录：`_kb/schema/` 或领域 `_kb/schema/`
- 用途：保存结构规则、模板、分类规则、操作手册

## 标准流程：从 clipping 到正式知识页

假设你用浏览器插件把一篇文章抓进了 `Clippings/`。

### 步骤 1：保留 clipping 原文

- 不要立刻删除
- 不要立刻复制到多个地方
- 先把它视为 inbox capture

### 步骤 2：判断知识域

判断标准：

- 如果核心是“工具怎么安装、怎么配置、怎么使用”，通常进 `Tools/`
- 如果核心是“概念、方法论、架构、工程模式、系统设计”，通常进 `Tech/`
- 如果跨多个领域，再考虑 root `_kb/wiki/` 的跨域 hub

### 步骤 3：提升为 raw

最佳实践不是把全文复制进 `_kb/raw/`，而是：

1. 在合适的 `_kb/raw/` 下创建一个 `*-source.md`
2. 写清来源 URL、作者、时间、原 clipping 路径、归档理由
3. 明确“这篇内容为什么属于这个领域”

只有在下面情况，才建议复制或移动全文原始资料进入 `_kb/raw/`：

- clipping 内容后续可能被覆盖或丢失
- 你要做可移植的领域资料包
- 原始资料不是 markdown clipping，而是 PDF、图片、代码快照等独立资产

### 步骤 4：生成或更新正式知识页

- 在对应领域的正式笔记目录里写 canonical note
- 例如 `Tech/Agent/`、`Tools/AI/`
- 优先更新已有主题页，避免重复

### 步骤 5：更新 index 和 log

- 更新领域 `_kb/index.md`
- 追加领域 `_kb/log.md`
- 如果内容有跨域意义，再决定是否更新 root `_kb/wiki/index.md` 或新增跨域 hub

### 步骤 6：必要时更新 schema

只有当下面情况发生时，才改 schema：

- 你引入了一个新的知识域或子域
- 你改变了正式笔记模板
- 你改变了来源提升策略
- 你改变了索引或日志维护规则

## 如何选择目录

选择顺序固定为：

1. 先判断领域
2. 再判断它是 raw 还是 wiki
3. 最后判断要不要改 schema

快速判断：

- 原始采集：`Clippings/`
- 领域来源记录：`<Domain>/_kb/raw/`
- 跨域来源记录：`_kb/raw/`
- 正式笔记：现有领域内容目录
- 规则文档：`_kb/schema/` 或 `<Domain>/_kb/schema/`

## 什么时候写 raw

建议写 raw 的情况：

- 资料有长期参考价值
- 后续会被多篇笔记引用
- 你需要保存来源链和归档理由
- 你需要把一篇 inbox clipping 正式纳入知识库

不建议每次都写 raw 的情况：

- 只是短期灵感
- 没有后续整理价值
- 与已有来源高度重复

## 什么时候改 schema

schema 不跟着每篇新笔记一起膨胀。

应该改 schema 的典型触发器：

- 新知识域出现
- 新笔记模板定稿
- 新的 intake / promotion / lint 规则形成共识
- 你发现多个会话里都在重复解释同一条归档规则

## 什么时候更新 index 和 log

### index

在以下情况更新：

- 新增 canonical note
- 某个主题页成为该领域关键入口
- 某类内容的分类方式发生变化

### log

在以下情况更新：

- 一次正式 ingest 完成
- 一次大规模重构完成
- 一次 schema 规则发生重要变更

## 推荐日常工作流

最稳的日常节奏：

1. 平时只管往 `Clippings/` 收资料
2. 定期挑选高价值内容做“提升”
3. 每次只处理一个主题域
4. 先建 raw source record
5. 再写 canonical note
6. 再更新 index 和 log
7. 最后跑 lint

## 推荐提示词模板

### 从 clipping 提升为领域知识

```text
Use $llm-wiki-curator to ingest `Clippings/<file>.md`, decide the right knowledge domain, create a source record under that domain’s `_kb/raw/`, create or update the canonical note in the domain note tree, and refresh the domain index and log.
```

### 把正式知识页整理成 Obsidian 风格

```text
Use $note-curator to rewrite this canonical note in my Obsidian style with `title`, `created`, `description`, `tags`, `## 目录`, `## Key Takeaways`, and `## 参考来源`.
```

### 当知识结构发生变化时更新 schema

```text
Use $llm-wiki-curator to update the relevant schema docs because this note introduces a new subdomain, template, or indexing rule.
```

## 常见误区

- 把 `Clippings/` 当正式知识库
- 每篇 clipping 都复制一份到 `_kb/raw/`
- 每来一篇新文章就新建一篇正式笔记，不做合并
- 把 schema 当成笔记目录索引
- 不维护 log，导致之后不知道哪些内容是“正式提升”过的

## 参考来源

- [[notes-vault-knowledge-base-architecture]]
- [[note-frontmatter-and-sections]]
- [[maintenance]]
