---
title: "操作日志"
created: 2026-06-28
---
tags: [meta, log]

# 操作日志

> 新条目加在最上面。表格是快速索引，详情在下方。

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-06-28 | 🔧 scaffold | 四合一原则重构 wiki：7 模板 + Home MOC + 4 领域 MOC |

---

## 详情

### 2026-06-28 · scaffold

**操作：** 初始化并优化 wiki 结构

**设计原则：** 渐进式摘要 + 模板即结构契约 + MOC 驱动 + 可扫读

**创建内容：**
- 7 个 `_templates/`：concept, howto, course, design, entity, source, question
- `wiki/overview.md`：Home MOC（知识版图入口）
- `wiki/hot.md`：跨会话热缓存
- `wiki/log.md`：本文件
- `wiki/index.md`：AI 检索用平面索引
- 4 个领域 MOC：learning, work, thinking, goals
- `.raw/` 来源文件夹：articles, transcripts, screenshots, data, assets
- `.obsidian/snippets/vault-colors.css`：wiki 文件夹彩色标记

**下一步：** 首次 ingest + 典范页面创建

## 旧体系日志（从 _kb/ 迁移）

> ⚠️ 以下为历史记录，引用的 _kb/ 目录已在 2026-06-28 迁移中删除。
> 来源记录移至 .raw/articles/，规则文档整合进 wiki/meta/ 和 AGENTS.md。

### _kb/log.md (vault-level)
# Wiki 维护日志

vault 级 ingest、query、lint 的追加式维护日志。

## [2026-05-07] seed | initialize knowledge base
- created `.raw/articles/`, `_kb/wiki/`, and `wiki/meta/` control-plane folders

## [2026-05-07] seed | karpathy llm wiki method
- added a raw source record for Karpathy's gist
- created the first compiled wiki page
- linked the page from `_kb/wiki/index.md`

## [2026-06-03] maintenance | 中文文件名规范更新
- renamed formal notes, schema long-form pages, wiki method pages, and promoted source records to Chinese filenames
- kept `index.md`, `log.md`, and `README.md` as stable control entry files
- added root `AGENTS.md` and `agent.md`
- synchronized naming rules in the note/wiki related skills

## [2026-06-10] ingest | Thinking 思维模型手册
- initialized `Thinking/_kb/index.md` and `Thinking/_kb/log.md`
- added `Thinking/Cognitive Upgrade/高杠杆思维模型实战手册.md`
- linked Thinking from `_kb/wiki/index.md`

### Tech/_kb/log.md
# Tech 领域日志

Tech 领域 ingest、query、lint 的追加式维护日志。

## [YYYY-MM-DD] seed | initialize domain knowledge base
- created _kb/raw and _kb/schema
- prepared domain index and domain log

## [2026-05-07] ingest | harness engineering article
- promoted the clipping into `Tech/.raw/articles/Harness Engineering 文章来源记录.md`
- created canonical note `Tech/Agent/AI Agent 的 Harness Engineering.md`
- updated `Tech/_kb/index.md` and `Tech/wiki/meta/Tech 领域分类.md`

## [2026-05-11] ingest | elasticsearch notion export
- promoted the original Notion export from `Temp/` into `Tech/.raw/articles/elasticsearch-notion-export/`
- added source record `Tech/.raw/articles/Elasticsearch Notion 导出来源记录.md`
- created the canonical Elasticsearch page set under `Tech/Backend/DateBase/elasticsearch/`
- updated `Tech/_kb/index.md` so the Tech domain can route directly to the Elasticsearch knowledge set

## [2026-06-03] maintenance | 中文文件名规范更新
- renamed Tech formal notes and source records to Chinese filenames
- updated `Tech/_kb/index.md`, `Tech/_kb/log.md`, and Tech schema links

## [2026-06-15] ingest | JavaGuide AI 应用开发知识体系
- added `Tech/AI/` as the AI 应用开发 formal note layer
- created source record `Tech/.raw/articles/JavaGuide AI 应用开发知识体系来源记录.md`
- created canonical note `Tech/AI/AI 应用开发学习体系.md`
- updated `Tech/_kb/index.md` and `Tech/wiki/meta/Tech 领域分类.md`

## [2026-06-21] ingest | Java 后端转 AI 应用开发学习路线
- preserved the original reading note `Inbox/Java-to-AI-roadMap - AI 应用开发与 Agent 学习路线.md`
- created source record `Tech/.raw/articles/JavaGuide Java Go 开发者 AI 应用开发路线来源记录.md`
- created the canonical workspace `Tech/AI/Java 转 AI 应用开发/` with a roadmap, progress checklist, and staged topic skeletons
- linked the previous `Tech/AI/AI 应用开发学习体系.md` overview to the new workspace
- updated `Tech/_kb/index.md` and `Tech/wiki/meta/Tech 领域分类.md`

## [2026-06-22] ingest | LLM 运行机制
- added source record `Tech/.raw/articles/JavaGuide LLM 运行机制来源记录.md`
- expanded the planned LLM fundamentals skeleton into `Tech/AI/Java 转 AI 应用开发/01-大模型基础/LLM 运行机制：Token、上下文窗口与采样参数.md`
- normalized time-sensitive model data into verification notes and added engineering decision tables, Mermaid flows, observability fields, and validation experiments
- updated the Java-to-AI workspace links after aligning the filename with the formal note title

## [2026-06-22] ingest | 大模型结构化输出
- added source record `Tech/.raw/articles/JavaGuide 大模型结构化输出来源记录.md`
- expanded and renamed the planned skeleton as `Tech/AI/Java 转 AI 应用开发/01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地.md`
- verified provider-specific boundaries against current OpenAI, Anthropic, Gemini, MCP, and JSON Schema documentation
- separated model-facing parameters from trusted runtime metadata and added Java validation, security, retry, observability, and rollout guidance
- updated all Java-to-AI workspace backlinks to the canonical filename

## [2026-06-22] ingest | Hello-Agents 第一章初识智能体
- moved the existing chapter note from `AI/AI Developer/Agent/` into the canonical Tech AI learning layer
- created `Tech/AI/Hello-Agents/index.md` as the 16-chapter course map and progress entry
- rewrote `第一章 初识智能体.md` with frontmatter, TOC, key takeaways, comparison tables, Mermaid diagrams, safer tool-call patterns, production boundaries, and evaluation guidance
- added source record `Tech/.raw/articles/Hello-Agents 第一章来源记录.md`
- linked the course and chapter from `Tech/_kb/index.md`

## [2026-06-22] ingest | 大模型提示词工程
- added source record `Tech/.raw/articles/JavaGuide 大模型提示词工程来源记录.md`
- expanded and renamed the planned Prompt skeleton as `Tech/AI/Java 转 AI 应用开发/02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？.md`
- restructured the topic around prompt anatomy, six technique families, evidence handling, evaluation, lifecycle governance, injection defense, and Agent context boundaries
- verified the Spring AI example against the current 2.0.0 reference and normalized reasoning-model guidance against current provider documentation
- updated all Java-to-AI workspace backlinks to the canonical filename

### Tools/_kb/log.md
# Tools 领域日志

## [2026-06-27] ingest | Superpowers 安装使用与 AI 编程工作流
- added source record `Tools/.raw/articles/Superpowers GitHub 来源记录.md`
- added canonical note `Tools/AI/Superpowers 安装使用与 AI 编程工作流实践指南.md`
- updated `Tools/_kb/index.md`

## [2026-06-06] ingest | macOS tmux 安装使用与 AI 研发工作流
- added canonical note `Tools/Mac/macOS tmux 安装使用与 AI 研发工作流最佳实践.md`
- updated `Tools/_kb/index.md`

## [2026-06-03] ingest | Spec Kit 与 SDD 实践指南
- added canonical note `Tools/AI/Spec Kit 与 SDD 规范驱动开发实践指南.md`
- updated `Tools/_kb/index.md`

## [2026-06-03] maintenance | 中文文件名规范更新
- renamed Tools formal notes and schema rules to Chinese filenames
- updated `Tools/_kb/index.md`, `Tools/_kb/log.md`, and `Tools/AI/Skills/README.md`

Append-only activity log for ingests, queries, and lint passes in this domain.

## [YYYY-MM-DD] seed | initialize domain knowledge base
- created _kb/raw and _kb/schema
- prepared domain index and domain log
