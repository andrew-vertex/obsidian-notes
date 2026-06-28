---
title: "claude-obsidian 快速上手"
created: 2026-06-28
description: "用 claude-obsidian 插件搭建 AI 驱动的 Obsidian 个人知识库"
tags: [claude-obsidian, obsidian, claude-code, wiki, second-brain]
tool: "claude-obsidian"
version: "1.9.2"
platform: mac
layer: 2
status: developing
---

# claude-obsidian 快速上手

> 适用场景：你已经在用 Obsidian 记笔记，想让 Claude 帮你自动组织、交叉引用、维护知识库。

## 核心要点

1. claude-obsidian 不是聊天插件，是知识引擎——Claude 主动创建、组织、维护笔记
2. 核心工作流：丢来源 → ingest → Claude 提取实体/概念 → 更新索引/缓存
3. 四合一模板体系（渐进式摘要 + 结构契约 + MOC 导航 + 可扫读）让笔记质量和一致性大幅提升
4. 跨会话记忆靠 `hot.md`：每次会话结束 Claude 自动更新，下次自动加载
5. 所有笔记是纯 Markdown 文件，你的数据永远在你手里

## 环境要求

- Claude Code（最新版）
- Obsidian v1.9.10+（支持 Bases）或 v1.6+（Dataview 回退）
- Git（用于 vault 自动提交）
- macOS / Linux / Windows

## 安装

```bash
# 1. 添加 marketplace
claude plugin marketplace add AgriciDaniel/claude-obsidian

# 2. 安装插件
claude plugin install claude-obsidian@agricidaniel-claude-obsidian

# 3. 验证
claude plugin list
```

## 配置

无需额外配置即可使用。可选增强：

- **CSS 彩色标记**：`.obsidian/snippets/vault-colors.css` 为 wiki 文件夹着色
- **MCP 直连**：配置后 Claude 可直接读写 vault 文件（可选，文件系统已可操作）
- **Obsidian Git 插件**：每 15 分钟自动提交 vault，防数据丢失

## 命令速查

| 命令 | 作用 | 示例 |
|------|------|------|
| `/wiki` | 初始化或继续维护 wiki | `/wiki` |
| `ingest [文件]` | 摄入来源，自动创建 8-15 个 wiki 页 | `ingest article.md` |
| `ingest all of these` | 批量摄入多个来源 | `ingest all of these` |
| `what do you know about X?` | 从 wiki 查询并综合回答 | `what do you know about RAG?` |
| `lint the wiki` | 健康检查：孤立页、死链、缺口 | `lint the wiki` |
| `/save` | 将当前对话保存为 wiki 笔记 | `/save` |
| `/autoresearch [主题]` | 自主 web 研究 → 综合 → 归档 | `/autoresearch MCP protocol` |
| `/think [问题]` | 10 原则深度思考框架 | `/think 如何设计核赔系统架构` |

## 最佳实践

> [!tip] 渐进摄入
> 不要一次丢 10 个来源。每次摄入 1-2 个，和 Claude 讨论关键收获后再继续。质量 > 数量。

> [!tip] 定期 lint
> 每 10-15 次摄入后运行 `lint the wiki`。孤立页和矛盾标注积攒多了很难清理。

> [!tip] hot.md 是你的跨会话记忆
> 确保会话结束时 hot.md 被更新。下次打开 Claude 会自动加载上下文，无需重新解释。

> [!tip] 模板是参考，不是模具
> 7 个模板的 section 顺序是建议。如果某个 notes 不需要对比表或 Mermaid，删掉即可。灵活使用。

## 常见问题

> [!warning] 摄入后页面太多？
> 正常。一个来源通常产生 8-15 个页面（来源摘要 + 实体页 + 概念页）。页面多说明提取充分，不是问题。

> [!warning] 不要直接修改 .raw/
> `.raw/` 是来源文件的只读仓库。要修改内容，去对应的 wiki 页面。

## 关联

- [[Tools/claude-obsidian 插件使用指南|完整使用指南]] — 详细命令参考、目录结构、方法论模式
- [[wiki/meta/2026-06-28-wiki-scaffold-design|Scaffold 设计文档]] — 本 vault 结构的设计决策
- [[高杠杆思维模型]] — 费曼学习法、80/20 等思维模型在知识管理中的应用

## 参考来源

- [claude-obsidian GitHub](https://github.com/AgriciDaniel/claude-obsidian)
- [深度解析博客](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)
