---
title: "claude-obsidian 插件使用指南"
type: howto
created: 2026-06-28
updated: 2026-06-29
description: "Claude Code 的 Obsidian 知识库管理插件：从安装、配置到完整工作流的操作手册"
tags:
  - obsidian
  - claude-code
  - plugin
  - wiki
  - second-brain
  - pkm
  - knowledge-management
tool: "claude-obsidian"
version: "v1.9.2"
platform: mac
layer: 2
status: developing
aliases:
  - claude-obsidian
sources:
  - "https://github.com/AgriciDaniel/claude-obsidian"
---

# claude-obsidian 插件使用指南

> **一句话适用场景**：你希望 Claude 自动管理 Obsidian 知识库——放入来源材料，Claude 自动提取实体和概念、创建交叉引用的 wiki 页面、维护索引和热缓存，你随时用自然语言查询。

## 核心要点

1. **核心循环**：`/wiki` 初始化 → `ingest` 建库 → `what do you know about X?` 查询 → `lint the wiki` 体检 → `/save` 保存对话
2. **自动组织**：Claude 从来源中提取实体、概念、对比，自动创建 8-15 个交叉引用页面
3. **跨会话记忆**：`hot.md` 热缓存让新会话的 Claude 立刻知道上次在做什么
4. **四种方法论**：Generic（默认）、LYT（MOC 导航）、PARA（可操作性）、Zettelkasten（原子笔记）
5. **命令有明确顺序**：初始化 → 摄入 → 查询 → 维护，跳步骤会导致索引不完整
6. **Hooks 自动化**：SessionStart 自动加载 hot cache，PostToolUse 自动 git commit
7. **开源 MIT**：仓库 [AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)，版本 v1.9.2

## 目录

- [环境要求](#环境要求)
- [安装](#安装)
- [配置](#配置)
- [快速开始：首次使用全流程](#快速开始首次使用全流程)
- [命令速查](#命令速查)
- [典型工作流程](#典型工作流程)
- [目录结构](#目录结构)
- [方法论模式](#方法论模式-v18)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [与其他工具对比](#与其他-obsidian-ai-插件的区别)
- [推荐插件](#推荐插件)
- [卸载](#卸载)
- [关联](#关联)
- [参考来源](#参考来源)

---

## 环境要求

| 组件 | 最低版本 | 说明 |
|------|---------|------|
| Claude Code | latest | https://claude.com/claude-code |
| Obsidian | v1.9.10+（支持 Bases） | v1.6+ 可用 Dataview 回退 |
| Python | 3.10+ | 可选：检索管线 + 测试套件 |
| Bash | 4.0+（或 zsh） | 安装脚本 |
| Git | any | vault 自动提交 |

**可选依赖：**

| 依赖 | 用途 |
|------|------|
| ollama | `/wiki-retrieve` 本地重排序 |
| defuddle-cli | `/defuddle` 干净 web 提取 |
| Anthropic API key | `/wiki-retrieve` 上下文前缀层（需 `--allow-egress`） |
| Local REST API 插件 | REST API 方式 MCP |

---

## 安装

### 方式一：Claude Code 插件安装（推荐）

```bash
# Step 1: 添加 marketplace
claude plugin marketplace add AgriciDaniel/claude-obsidian

# Step 2: 安装插件
claude plugin install claude-obsidian@agricidaniel-claude-obsidian

# Step 3: 验证安装
claude plugin list
```

安装完成后，在任何 Claude Code 会话中输入 `/wiki` 即可开始。

### 方式二：克隆为独立 Vault

```bash
git clone https://github.com/AgriciDaniel/claude-obsidian
cd claude-obsidian
bash bin/setup-vault.sh
```

在 Obsidian 中：**Manage Vaults → Open folder as vault → 选择 `claude-obsidian/`**。

在 Claude Code 中打开同一文件夹，输入 `/wiki`。

### 方式三：集成到已有 Vault

将 `WIKI.md` 复制到你的 vault 根目录，粘贴到 Claude 中：

```
Read WIKI.md in this project. Then:
1. Check if Obsidian is installed...
2. Check if the Local REST API plugin is running...
3. Configure the MCP server.
4. Ask me ONE question: "What is this vault for?"
Then scaffold the full wiki structure.
```

---

## 配置

### MCP 设置（可选）

MCP 让 Claude 可以直接读写 vault 笔记，无需复制粘贴。

**方式 A（基于 REST API）：**

```bash
# 1. 在 Obsidian 中安装 Local REST API 插件
# 2. 复制 API key
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "uvx",
  "args": ["mcp-obsidian"],
  "env": {
    "OBSIDIAN_API_KEY": "your-key",
    "OBSIDIAN_HOST": "127.0.0.1",
    "OBSIDIAN_PORT": "27124",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}' --scope user
```

**方式 B（基于文件系统，无需插件）：**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

### Hooks 系统

插件内置 4 个生命周期 Hooks，安装后自动生效：

| Hook 事件 | 类型 | 作用 | 触发时机 |
|-----------|------|------|---------|
| `SessionStart` | command + prompt | 自动加载 `wiki/hot.md` 到上下文 | 每次新会话开始 |
| `PostCompact` | prompt | 上下文压缩后重新加载 hot cache | 对话过长触发压缩后 |
| `PostToolUse` | command | Write/Edit 后自动 git commit `wiki/` 和 `.raw/` | 每次写入文件后 |
| `Stop` | prompt | 会话结束时提醒更新 `wiki/hot.md` | 会话结束 |

### 跨项目知识库

在其他项目的 `CLAUDE.md` 中添加以下内容，即可共享同一 vault：

```markdown
## Wiki Knowledge Base
Path: ~/path/to/vault

When you need context not already in this project:
1. Read wiki/hot.md first (recent context cache)
2. If not enough, read wiki/index.md
3. If you need domain details, read the relevant domain sub-index
4. Only then drill into specific wiki pages

Do NOT read the wiki for general coding questions or tasks unrelated to [domain].
```

---

## 快速开始：首次使用全流程

以下是**完整的首次使用命令序列**，按顺序执行即可完成从零到可用的搭建：

| 序号 | 命令/操作 | 说明 | 预计耗时 |
|:----:|------|------|:----:|
| 1 | `claude plugin install claude-obsidian@agricidaniel-claude-obsidian` | 安装插件（只需一次） | 1 min |
| 2 | 在 Claude Code 中打开你的 Obsidian vault 目录 | 让 Claude 能访问你的 vault 文件 | — |
| 3 | `/wiki` | 启动 wiki 模式。Claude 会检查 Obsidian 状态和 MCP 配置 | 30s |
| 4 | 回答 Claude 的问题："What is this vault for?" | 描述用途，例如"个人知识库/技术学习/项目文档" | 1 min |
| 5 | Claude 自动搭建 wiki 结构 | 创建 `wiki/` 目录、`hot.md`、`index.md`、`log.md`、`overview.md` | 2-3 min |
| 6 | 放入第一个来源文件到 `.raw/` | 例如：将一篇技术文章保存到 `.raw/articles/` | — |
| 7 | `ingest .raw/articles/你的文件.md` | 让 Claude 摄入第一篇内容，创建 8-15 个 wiki 页面 | 3-5 min |
| 8 | `what do you know about [刚摄入的主题]?` | 验证查询能力——Claude 应综合回答并带 `[[引用]]` | 30s |
| 9 | `lint the wiki` | 第一次健康检查，看看索引是否完整 | 1 min |
| 10 | `/save session` | 保存本次会话摘要到 wiki | 30s |

> [!tip] 首次使用建议
> 前 3-5 次摄入后，每次都要 `lint the wiki` 检查。等 wiki 积累到 30+ 页面后，可以降低 lint 频率到每 5 次摄入一次。

### 启动后的检查清单

每次 `/wiki` 启动后，确认以下状态：

| 检查项 | 正常状态 | 异常处理 |
|--------|---------|---------|
| hot.md 加载 | Claude 提到"上次我们在讨论 X" | 如果没加载，手动 `Read wiki/hot.md` |
| index.md 存在 | `wiki/index.md` 有内容 | 如果没有，说 `rebuild the index` |
| .raw/ 有新文件 | 有未处理的来源 | 提醒 Claude：`there are new files in .raw/` |

---

## 命令速查

### 命令总览（按使用阶段排列）

| 阶段 | 命令 | 功能 | 示例 |
|:----:|------|------|------|
| 🔰 初始化 | `/wiki` | 初始化/检查 vault 设置，从 hot cache 恢复上下文 | `/wiki` |
| 📥 摄入 | `ingest [file]` | 摄入单个来源，提取实体/概念，创建 8-15 个页面 | `ingest article.md` |
| 📥 摄入 | `ingest all of these` | 批量摄入多个来源，最后统一交叉引用 | `ingest all of these` |
| 📥 摄入 | `ingest everything in .raw/` | 摄入 .raw/ 中所有未处理文件 | `ingest everything in .raw/` |
| 🔍 查询 | `what do you know about X?` | 从 wiki 综合查询，带 `[[引用]]` | `what do you know about transformer?` |
| 🔍 查询 | `find pages about X` | 搜索相关 wiki 页面（不综合回答） | `find pages about microservices` |
| 🔍 查询 | `show me the index` | 查看 wiki 索引概览 | `show me the index` |
| 🔧 维护 | `lint the wiki` | 8 类健康检查：孤立页、死链、缺口、矛盾等 | `lint the wiki` |
| 🔧 维护 | `update hot cache` | 手动刷新 `hot.md` | `update hot cache` |
| 🔧 维护 | `rebuild the index` | 重建 `wiki/index.md` | `rebuild the index` |
| 💾 保存 | `/save` | 分析对话，保存最有价值内容为 wiki 笔记 | `/save` |
| 💾 保存 | `/save [name]` | 用指定标题保存（跳过命名确认） | `/save transformer 架构分析` |
| 💾 保存 | `/save session` | 保存完整会话摘要 | `/save session` |
| 💾 保存 | `/save concept [name]` | 显式保存为概念页 | `/save concept 注意力机制` |
| 💾 保存 | `/save decision [name]` | 显式保存为决策记录 | `/save decision 选用 Milvus` |
| 🧠 研究 | `/autoresearch [topic]` | 3 轮自主 web 研究循环，创建研究报告 | `/autoresearch RAG 最新进展` |
| 🧠 研究 | `/autoresearch` | 若启用 DragonScale，显示 vault 前沿候选话题 | `/autoresearch` |
| 🎨 画布 | `/canvas` | 状态检查——报告节点数、列出区域 | `/canvas` |
| 🎨 画布 | `/canvas new [name]` | 创建新画布 | `/canvas new 架构图` |
| 🎨 画布 | `/canvas add image [path]` | 添加图片到画布（支持 URL/本地路径） | `/canvas add image diagram.png` |
| 🎨 画布 | `/canvas add text [content]` | 添加 Markdown 文本卡片 | `/canvas add text "核心流程"` |
| 🎨 画布 | `/canvas add note [page]` | 将 wiki 页面固定为链接卡片 | `/canvas add note transformer` |
| 🎨 画布 | `/canvas zone [name] [color]` | 添加标签区域 | `/canvas zone 输入层 blue` |
| 🎨 画布 | `/canvas from banana` | 捕获最近生成的 AI 图片 | `/canvas from banana` |
| 🤔 思考 | `/think [problem]` | 应用 10 原则思考框架分析问题 | `/think 如何设计缓存策略` |

### 命令使用顺序建议

```
┌────────────────────────────────────────────────────────────┐
│  阶段 1: 初始化（只需一次）                                  │
│  /wiki  →  回答 vault 用途  →  Claude 搭建结构              │
├────────────────────────────────────────────────────────────┤
│  阶段 2: 知识构建（循环）                                    │
│  放入来源到 .raw/  →  ingest [file]  →  /save [insight]    │
│       ↑                                          │         │
│       └────────── 重复直到覆盖领域 ───────────────┘         │
├────────────────────────────────────────────────────────────┤
│  阶段 3: 查询利用                                           │
│  /wiki  →  what do you know about X?  →  /save [answer]    │
├────────────────────────────────────────────────────────────┤
│  阶段 4: 定期维护                                           │
│  lint the wiki  →  修复问题  →  update hot cache            │
└────────────────────────────────────────────────────────────┘
```

---

## 典型工作流程

### 1. 每日使用循环（完整命令序列）

下表是**一次完整会话**的理想命令顺序。实际使用中可按需跳步：

| 步骤 | 命令 | 说明 | 频率 |
|:----:|------|------|:----:|
| 1 | `/wiki` | 启动 wiki 模式。Claude 自动加载 `hot.md`，恢复上次上下文 | 每次会话 |
| 2 | 检查 `.raw/` | 看是否有新来源文件待摄入 | 每次会话 |
| 3 | `ingest [新文件]` | 摄入新来源——这是知识增长的核心操作 | 有新文件时 |
| 4 | 与 Claude 讨论收获 | Claude 会在摄入后与你讨论关键要点 | 每次摄入后 |
| 5 | `what do you know about X?` | 针对当前工作查询相关知识 | 按需 |
| 6 | `/save [insight]` | 将对话中有价值的洞察保存为 wiki 笔记 | 有洞察时 |
| 7 | `lint the wiki` | 健康检查（建议每 5 次摄入或每天一次） | 定期 |
| 8 | `/save session` | 会话结束时保存摘要，更新 hot cache | 每次会话结束 |

### 2. 摄入流程（Claude 内部执行 11 步）

当你输入 `ingest [file]` 时，Claude 内部按以下顺序执行：

| 步骤 | 操作 | 产出 |
|:----:|------|------|
| 1 | 完整读取来源文件 | 理解全部内容 |
| 2 | 与你讨论关键收获 | 确认理解正确（除非你说"直接摄入"） |
| 3 | 在 `wiki/sources/` 创建来源摘要页 | 1 个来源页，包含元数据和要点 |
| 4 | 为每个人物/组织/产品创建或更新实体页 | N 个 `wiki/entities/` 页面 |
| 5 | 为重要想法创建或更新概念页 | M 个 `wiki/concepts/` 页面 |
| 6 | 如有对比关系，创建对比页 | `wiki/comparisons/` 页面 |
| 7 | 更新相关领域页和 `_index.md` | 领域索引保持最新 |
| 8 | 如果大局有变，更新 `overview.md` | 执行摘要同步更新 |
| 9 | 更新 `index.md` | 全局索引包含新页面 |
| 10 | 更新 `hot.md` | 热缓存反映最新操作 |
| 11 | 追加 `log.md`（新条目在最上面） | 操作日志可追溯 |
| 🔍 | **检查矛盾**，用 `[!contradiction]` 标注 | 知识一致性保障 |

> [!tip] 批量摄入
> 多个来源建议用 `ingest all of these`：Claude 会逐个处理，最后统一交叉引用。比一个一个 ingest 更高效，因为交叉引用只在最后做一次。

### 3. 查询流程（Claude 内部执行 6 步）

当你输入 `what do you know about X?` 时：

| 步骤 | 操作 | 说明 |
|:----:|------|------|
| 1 | 读取 `wiki/hot.md` | 热缓存可能直接有答案 |
| 2 | 读取 `wiki/index.md` | 找到相关页面列表 |
| 3 | 读取 3-5 个最相关页面 | 不超过 10 个，避免上下文过载 |
| 4 | 综合回答 | 用 `[[wikilink]]` 格式引用来源页面 |
| 5 | 主动提出保存 | "要不要把这个答案保存为 wiki/question 页？" |
| 6 | 标记知识缺口 | 如果发现 wiki 中缺乏某方面信息，主动告知 |

### 4. 维护工作流

```
lint the wiki
    │
    ├── 孤立页 → 人工判断：链接到相关页面或删除
    ├── 死链   → 更新或移除损坏的 [[wikilink]]
    ├── 缺口   → 标记需补充的领域，后续 ingest 补充
    ├── 矛盾   → 检查 [!contradiction] 标记，人工裁决
    ├── 陈旧   → 更新过时内容，刷新 hot cache
    └── 索引   → rebuild the index 确保完整性
```

| Lint 检查项 | 说明 | 处理方式 |
|------------|------|---------|
| 孤立页 | 没有任何入链的页面 | 决定是链接还是删除 |
| 死链 | `[[指向不存在页面]]` | 更新链接或创建目标页 |
| 知识缺口 | 领域覆盖不完整 | 找来源补充 ingest |
| 矛盾标记 | `[!contradiction]` 标注 | 人工判断哪方正确 |
| 陈旧内容 | 过时信息 | 更新或标记 `status: archived` |
| 索引完整性 | index.md 是否包含所有页面 | `rebuild the index` |
| 空页面 | 只有 frontmatter 没有内容 | 填充内容或删除 |
| 坏 frontmatter | 缺少必要字段 | 补全 `title`/`created`/`tags` |

---

## 目录结构

### 插件结构

```
claude-obsidian/
├── .claude-plugin/
│   ├── plugin.json                 # 插件清单
│   └── marketplace.json            # 分发配置
├── skills/                          # 15 个 Claude Code 技能 (v1.9.2)
│   ├── wiki/                        # 主协调器 + 参考文档
│   ├── wiki-ingest/                 # 来源摄入
│   ├── wiki-query/                  # 从 vault 回答问题
│   ├── wiki-lint/                   # vault 健康检查
│   ├── wiki-cli/                    # Obsidian CLI 传输层 (v1.7+)
│   ├── wiki-retrieve/               # 混合检索 (v1.7+, 可选)
│   ├── wiki-mode/                   # 方法论模式路由 (v1.8+)
│   ├── wiki-fold/                   # 日志折叠 (DragonScale 可选)
│   ├── save/                        # /save: 保存对话到 wiki
│   ├── autoresearch/                # 自主研究循环
│   ├── canvas/                      # 可视层 (图片/PDF/笔记)
│   ├── defuddle/                    # web 提取包装器
│   ├── obsidian-bases/              # Bases schema 参考
│   ├── obsidian-markdown/           # OFM 语法参考
│   └── think/                       # 10 原则思考框架 (v1.9+)
├── agents/
│   ├── verifier.md                  # 提交前审计 agent (v1.7.1+)
│   ├── wiki-ingest.md               # 并行批量摄入 agent
│   └── wiki-lint.md                 # 健康检查 agent
├── commands/                        # 斜杠命令入口
├── hooks/
│   └── hooks.json                   # SessionStart + Stop + PostToolUse hooks
├── scripts/                         # 12 个辅助脚本
├── tests/                           # 9 个密封测试套件 (~1240 断言)
├── bin/                             # 5 个安装脚本
├── _templates/                      # Obsidian Templater 模板
├── wiki/                            # 种子 vault 内容
├── docs/                            # 指南 + 审计报告
├── WIKI.md                          # 完整 schema 参考
├── CLAUDE.md                        # 项目说明
└── README.md
```

### Vault 内结构（Wiki 模式）

```
vault/
├── .raw/                   # 不可变来源文件（只读）
│   ├── articles/           # 文章
│   ├── transcripts/        # 转录
│   ├── screenshots/        # 截图
│   ├── data/               # 数据文件
│   └── assets/             # 资源
├── wiki/                   # LLM 生成的知识库
│   ├── index.md             # 所有 wiki 页面的主目录
│   ├── log.md               # 操作日志（追加模式，最新在上）
│   ├── hot.md               # 热缓存（~500 字，跨会话记忆）
│   ├── overview.md          # 整个 wiki 的执行摘要
│   ├── sources/             # 每个来源一页摘要
│   ├── entities/            # 人物、组织、产品
│   ├── concepts/            # 想法、模式、框架
│   ├── domains/             # 顶级主题领域
│   ├── comparisons/         # 对比分析
│   ├── questions/           # 问题与回答
│   └── meta/                # 仪表板、lint 报告
├── _templates/              # Templater 模板
├── _attachments/            # 图片和 PDF
└── WIKI.md                  # Schema 参考
```

---

## 方法论模式 (v1.8+)

四种组织哲学，通过 `bash bin/setup-mode.sh` 选择：

| 模式 | 哲学 | 归档约定 | 适用场景 |
|------|------|---------|---------|
| **Generic**（默认） | 无特定观点，保留 v1.7 行为 | `wiki/sources/`, `wiki/entities/`, `wiki/concepts/` | 通用知识库，不确定选哪个时 |
| **LYT** | 笔记链接，文件夹不重要，MOC 是导航原语 | `wiki/mocs/<topic>-moc.md` + `wiki/notes/<atomic-note>.md` | 注重想法连接，自由探索 |
| **PARA** | 按可操作性组织（Projects, Areas, Resources, Archives） | `wiki/projects/`, `wiki/areas/`, `wiki/resources/`, `wiki/archives/` | 行动导向，项目驱动 |
| **Zettelkasten** | 原子笔记，唯一 ID，密集双向链接，无文件夹 | `wiki/<YYYYMMDDHHMMSSffffff>-<slug>.md`（扁平，时间戳） | 学术研究，深度思考 |

### Vault 用途场景

| 场景 | 适用情况 | 推荐方法论 |
|------|---------|:----:|
| **Website** | 网站地图、内容审计、SEO wiki | Generic |
| **GitHub** | 代码库映射、架构 wiki | Generic / LYT |
| **Business** | 项目 wiki、竞争情报 | PARA |
| **Personal** | 第二大脑、目标、日志综合 | LYT / PARA |
| **Research** | 论文、概念、论文写作 | Zettelkasten |
| **Book/Course** | 章节追踪、课程笔记 | Generic / Zettelkasten |

---

## 最佳实践

### 摄入策略

| 实践 | 说明 |
|------|------|
| 🟢 **先少后多** | 前 3 次只摄入 1-2 个来源，熟悉流程后再批量 |
| 🟢 **讨论收获** | 摄入后与 Claude 讨论关键收获——不要跳过第 2 步 |
| 🟢 **标注来源** | 在 `.raw/` 中使用子文件夹分类（articles/ transcripts/ screenshots/） |
| 🟢 **同类一起** | 相关内容一次批量 `ingest all of these`，交叉引用效果更好 |
| 🟡 **限制单次量** | 一次最多摄入 3-5 个长文来源，过多会稀释关注点 |
| 🔴 **不要跳过 lint** | 每 5 次摄入后必须 lint，否则孤立页和死链会悄悄积累 |

### 查询策略

| 实践 | 说明 |
|------|------|
| 🟢 **先广后细** | 先用 `what do you know about X?` 看全景，再追问细节 |
| 🟢 **追问到底** | Claude 回答后可以连续追问，直到满意 |
| 🟢 **保存答案** | 有洞察的 Q&A 用 `/save question [topic]` 保存 |
| 🟢 **利用 hot cache** | 下次 `/wiki` 时，Claude 会记得你在研究什么 |

### 维护节奏

| 频率 | 操作 | 说明 |
|:----:|------|------|
| 每次会话开始 | `/wiki` | 自动加载 hot cache，了解上下文 |
| 每次摄入后 | 讨论收获 → `/save [insight]` | 固化知识 |
| 每 5 次摄入 | `lint the wiki` | 健康检查 |
| 每周 | `update hot cache` | 手动刷新热点，确保跨会话连贯 |
| 每月 | 全面 lint + 手动审查 | 清理矛盾、更新过时内容 |

### 文件组织

| 实践 | 说明 |
|------|------|
| 🟢 `.raw/` 只放未处理文件 | 摄入完成后不要移动原始文件 |
| 🟢 使用子文件夹 | `.raw/articles/`、`.raw/transcripts/` 等，Claude 会根据路径推断类型 |
| 🟢 保留原始文件名 | 不要重命名 .raw/ 中的文件，方便追溯 |
| 🔴 不要手动编辑 wiki/ | wiki/ 下所有内容由 Claude 生成，手动编辑会与下一次 ingest 冲突 |

---

## 常见问题

### 安装与启动

| 问题 | 解决方案 |
|------|---------|
| `/wiki` 无响应 | 检查 `claude plugin list` 确认已安装；尝试重启 Claude Code |
| "Obsidian not found" | 确保 Obsidian 已安装且至少打开过一次 vault |
| MCP 连接失败 | 检查 Local REST API 插件是否运行，API key 是否正确 |

### 摄入与查询

| 问题 | 解决方案 |
|------|---------|
| 摄入后 index 没更新 | 手动 `rebuild the index` |
| 查询结果不相关 | 尝试更具体的表述；或用 `find pages about X` 先看有什么 |
| 摄入大文件超时 | 将大文件拆分为多个小文件分批 ingest |
| `[!contradiction]` 太多 | 正常现象——说明 wiki 在主动检测不一致。人工裁决并更新 |

### 维护

| 问题 | 解决方案 |
|------|---------|
| 孤立页太多 | `lint the wiki` → 决定每个孤立页的去留 |
| hot.md 没更新 | 手动 `update hot cache`；检查 Stop hook 是否正常 |
| git commit 太多 | PostToolUse hook 每次写入都 commit，正常。可调整 hook 配置 |

### 方法论切换

| 问题 | 解决方案 |
|------|---------|
| 可以中途切换模式吗？ | 可以，运行 `bash bin/setup-mode.sh` 重新选择。已有页面不会丢失 |
| 切换后索引不对 | `rebuild the index` 让 Claude 按新模式重新组织 |

---

## 与其他 Obsidian AI 插件的区别

| 能力 | claude-obsidian | Smart Connections | Copilot |
|------|:-:|:-:|:-:|
| 自动组织笔记 | ✅ | ❌ | ❌ |
| 矛盾标记 (`[!contradiction]`) | ✅ | ❌ | ❌ |
| 会话记忆（hot cache 跨会话） | ✅ | ❌ | ❌ |
| Vault 健康检查（8 类 lint） | ✅ | ❌ | ❌ |
| 自主研究循环（3 轮 web 研究） | ✅ | ❌ | ❌ |
| 方法论模式（LYT/PARA/Zettelkasten） | ✅ | ❌ | ❌ |
| 多模型支持 | ✅ Claude, Gemini, Codex, Cursor | ❌ 仅 Claude | ✅ 多 |
| 可视画布 | ✅ 通过 claude-canvas | ❌ | ❌ |
| 多写入安全（文件锁 v1.7+） | ✅ | ❌ | ❌ |
| 开源 | ✅ MIT | ✅ MIT | ⚠️ Freemium |

---

## 推荐插件

### Obsidian 社区插件

| 插件 | 用途 | 必要程度 |
|------|------|:----:|
| **Templater** | 从 `_templates/` 自动填充 frontmatter | 推荐 |
| **Obsidian Git** | 每 15 分钟自动提交 vault | 推荐 |
| **Calendar** | 右侧日历面板 | 可选 |
| **Thino** | 快速备忘捕获 | 可选 |
| **Excalidraw** | 手绘画布 | 可选 |
| **Banners** | Notion 风格标题图片 | 可选 |
| **Dataview**（旧版） | 仅用于旧版 `dashboard.md` 查询 | 可选 |

### 浏览器扩展

- **[Obsidian Web Clipper](https://obsidian.md/clipper)**: 一键发送网页到 `.raw/`

### CSS Snippets

`setup-vault.sh` 自动启用三个 snippet：

| Snippet | 效果 |
|---------|------|
| `vault-colors` | 文件浏览器中按类型着色 wiki 文件夹 |
| `ITS-Dataview-Cards` | 将 Dataview 表格转为视觉卡片网格 |
| `ITS-Image-Adjustments` | 精确控制图片尺寸（`\|100`） |

---

## 扩展：claude-canvas

如需更强大的可视层，可额外安装 [claude-canvas](https://github.com/AgriciDaniel/claude-canvas)：

```bash
claude plugin install AgriciDaniel/claude-canvas
```

提供 12 种模板、6 种布局算法、AI 图像生成、演示文稿和完整的画布编排功能，与 claude-obsidian vault 自动兼容。

---

## 卸载

```bash
# 插件方式安装的卸载
claude plugin uninstall claude-obsidian@agricidaniel-claude-obsidian
claude plugin marketplace remove AgriciDaniel/claude-obsidian

# Clone 方式安装的卸载
rm -rf /path/to/claude-obsidian
```

> vault 内容（`wiki/` 下）是纯 Markdown 文件，卸载后依然保留。

---

## 关联

- [[Karpathy LLM Wiki 知识库方法]] — 底层方法论：来源摄入 → LLM 提取实体/概念 → 交叉引用 wiki
- [[claude-obsidian 快速上手]] — 简化的快速上手指南
- [[wiki-scaffold-design]] — 本 vault 的 scaffold 设计文档（模板系统、分层策略、目录约定）

---

## 参考来源

- [GitHub 仓库](https://github.com/AgriciDaniel/claude-obsidian)
- [深度解析博客](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)
- [YouTube 演示](https://www.youtube.com/watch?v=a2hgayvr-H4)
- [Karpathy LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [claude-canvas 扩展](https://github.com/AgriciDaniel/claude-canvas)
