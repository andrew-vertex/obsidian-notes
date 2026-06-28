---
type: concept
title: "claude-obsidian 插件使用指南"
created: 2026-06-28
updated: 2026-06-28
tags:
  - obsidian
  - claude-code
  - plugin
  - wiki
  - second-brain
  - pkm
status: developing
aliases:
  - claude-obsidian
sources:
  - "https://github.com/AgriciDaniel/claude-obsidian"
---

# claude-obsidian 插件使用指南

## 概述

**claude-obsidian** 是一个 Claude Code 插件，将 Claude 变成你的 Obsidian 知识库的 AI 管理员。它基于 [Andrej Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)，实现了一个**自我组织的 AI 第二大脑**。

> 核心思路：你丢入来源（文章、视频、论文），Claude 自动提取实体和概念，创建交叉引用的 wiki 页面，维护索引和热缓存。知识像复利一样增长。

**仓库**: [AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) | **版本**: v1.9.2 | **许可证**: MIT

---

## 与传统 Obsidian AI 插件的区别

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

## 安装方式

### 方式一：作为 Claude Code 插件安装（推荐）

```bash
# Step 1: 添加 marketplace
claude plugin marketplace add AgriciDaniel/claude-obsidian

# Step 2: 安装插件
claude plugin install claude-obsidian@agricidaniel-claude-obsidian

# 验证安装
claude plugin list
```

在任何 Claude Code 会话中输入 `/wiki` 即可开始。

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

## 目录结构

```
claude-obsidian/
├── .claude-plugin/
│   ├── plugin.json                 # 插件清单
│   └── marketplace.json             # 分发配置
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

**Vault 内结构（Wiki 模式）：**

```
vault/
├── .raw/                   # 不可变来源文件（只读）
│   ├── articles/
│   ├── transcripts/
│   ├── screenshots/
│   ├── data/
│   └── assets/
├── wiki/                   # LLM 生成的知识库
│   ├── index.md             # 所有 wiki 页面的主目录
│   ├── log.md               # 操作日志（追加模式）
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

## 命令参考表

### 核心命令

| 命令 | 功能 | 示例 |
|------|------|------|
| `/wiki` | 初始化/检查 vault 设置，继续上次工作 | `/wiki` |
| `ingest [file]` | 摄入单个来源文件，创建 8-15 个 wiki 页面 | `ingest article.md` |
| `ingest all of these` | 批量处理多个来源，最后交叉引用 | `ingest all of these` |
| `what do you know about X?` | 从 wiki 中查询并综合回答，带引用 | `what do you know about machine learning?` |
| `lint the wiki` | 健康检查：孤立页、死链、缺口、建议 | `lint the wiki` |
| `update hot cache` | 刷新 hot.md 热缓存 | `update hot cache` |

### `/save` 命令

| 命令 | 功能 |
|------|------|
| `/save` | 分析整个对话，保存最有价值的内容为 wiki 笔记 |
| `/save [name]` | 用指定标题保存（跳过命名问题） |
| `/save session` | 保存完整的会话摘要 |
| `/save concept [name]` | 显式保存为概念页 |
| `/save decision [name]` | 显式保存为决策记录 |

### `/autoresearch` 命令

| 命令 | 功能 |
|------|------|
| `/autoresearch [topic]` | 对特定主题运行自主研究循环 |
| `/autoresearch` | 如果设置了 DragonScale，显示 vault 前沿候选话题 |

### `/canvas` 命令

| 命令 | 功能 |
|------|------|
| `/canvas` | 状态检查 — 报告节点数、列出区域 |
| `/canvas new [name]` | 创建新画布 |
| `/canvas add image [path]` | 添加图片到画布（支持 URL/本地路径） |
| `/canvas add text [content]` | 添加 Markdown 文本卡片 |
| `/canvas add pdf [path]` | 添加 PDF 文档节点 |
| `/canvas add note [page]` | 将 wiki 页面固定为链接卡片 |
| `/canvas zone [name] [color]` | 添加标签区域 |
| `/canvas list` | 列出所有画布及节点数 |
| `/canvas from banana` | 捕获最近生成的 AI 图片 |

### `/think` 命令

| 命令 | 功能 |
|------|------|
| `/think [problem]` | 应用 10 原则思考框架分析问题 |

---

## 典型工作流程

### 1. 首次使用（安装后）

```
0. 安装插件 → claude plugin install claude-obsidian@agricidaniel-claude-obsidian
1. 启动 Claude Code → 输入 /wiki
2. Claude 检查 Obsidian 是否安装
3. Claude 检查 MCP 配置状态
4. Claude 提问："What is this vault for?"
5. 描述用途（如：个人知识库 / 项目文档 / 研究课题）
6. Claude 自动搭建完整 wiki 结构
7. Claude 创建 hot.md, index.md, log.md, overview.md
8. Claude 建议进行第一次 ingest
```

### 2. 日常使用循环

```
┌──────────────────────────────────────┐
│  1. 放入来源 → .raw/ 文件夹           │
│  2. 告诉 Claude → "ingest [file]"    │
│  3. Claude 读取 → 提取实体/概念       │
│  4. Claude 创建/更新 → 8-15 个页面    │
│  5. Claude 更新 → index/log/hot      │
│  6. 提出问题 → "what do you know..." │
│  7. Claude 读取 hot → index → 页面   │
│  8. Claude 综合回答（带引用）         │
│  9. 定期 → "lint the wiki"            │
│  10. 会话结束 → /save 保存对话        │
└──────────────────────────────────────┘
```

### 3. 查询流程（Claude 内部执行顺序）

```
1. 读取 wiki/hot.md（热缓存，可能已有答案）
2. 读取 wiki/index.md（找到相关页面）
3. 读取 3-5 个最相关页面（不超过 10 个）
4. 综合回答，使用 [[wikilink]] 引用
5. 提供将答案保存为 wiki/question 页
6. 如果发现知识缺口，主动告知
```

### 4. 摄入流程（单个来源，Claude 内部执行步骤）

```
1. 完整读取来源文件
2. 与用户讨论关键收获（除非用户说"直接摄入"）
3. 在 wiki/sources/ 创建来源摘要页
4. 为每个提及的人物/组织/产品 创建/更新实体页
5. 为重要想法 创建/更新概念页
6. 更新相关领域页和 _index.md
7. 如果大局有变，更新 overview.md
8. 更新 index.md
9. 更新 hot.md
10. 追加 log.md（新条目放在最上面）
11. 检查矛盾，用 [!contradiction] 标注
```

---

## 方法论模式 (v1.8+)

四种组织哲学，通过 `bash bin/setup-mode.sh` 选择：

| 模式 | 哲学 | 归档约定 |
|------|------|---------|
| **Generic**（默认） | 无特定观点，保留 v1.7 行为 | `wiki/sources/`, `wiki/entities/`, `wiki/concepts/` |
| **LYT** | 笔记链接，文件夹不重要，MOC 是导航原语 | `wiki/mocs/<topic>-moc.md` + `wiki/notes/<atomic-note>.md` |
| **PARA** | 按可操作性组织（Projects, Areas, Resources, Archives） | `wiki/projects/`, `wiki/areas/`, `wiki/resources/`, `wiki/archives/` |
| **Zettelkasten** | 原子笔记，唯一 ID，密集双向链接，无文件夹 | `wiki/<YYYYMMDDHHMMSSffffff>-<slug>.md`（扁平，时间戳） |

---

## Vault 用途场景 (v1.0+)

| 场景 | 适用情况 |
|------|---------|
| **A: Website** | 网站地图、内容审计、SEO wiki |
| **B: GitHub** | 代码库映射、架构 wiki |
| **C: Business** | 项目 wiki、竞争情报 |
| **D: Personal** | 第二大脑、目标、日志综合 |
| **E: Research** | 论文、概念、论文写作 |
| **F: Book/Course** | 章节追踪、课程笔记 |

---

## Hooks 系统

插件内置 4 个生命周期 Hooks：

| Hook 事件 | 类型 | 作用 |
|-----------|------|------|
| `SessionStart` | command + prompt | 自动加载 `wiki/hot.md` 到上下文 |
| `PostCompact` | prompt | 上下文压缩后重新加载 hot cache |
| `PostToolUse` | command | Write/Edit 后自动 git commit wiki/ 和 .raw/ |
| `Stop` | prompt | 会话结束时提醒更新 `wiki/hot.md` |

---

## MCP 设置（可选）

MCP 让 Claude 可以直接读写 vault 笔记而无需复制粘贴。

**方式 A（基于 REST API）:**

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

**方式 B（基于文件系统，无需插件）:**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

---

## 跨项目知识库

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

## 推荐插件

### Obsidian 社区插件

| 插件 | 用途 |
|------|------|
| **Templater** | 从 `_templates/` 自动填充 frontmatter |
| **Obsidian Git** | 每 15 分钟自动提交 vault |
| **Calendar** | 右侧日历面板 |
| **Thino** | 快速备忘捕获 |
| **Excalidraw** | 手绘画布 |
| **Banners** | Notion 风格标题图片 |
| **Dataview**（可选，旧版） | 仅用于旧版 `dashboard.md` 查询 |

### 浏览器扩展

- **[Obsidian Web Clipper](https://obsidian.md/clipper)**: 一键发送网页到 `.raw/`

---

## CSS Snippets

`setup-vault.sh` 自动启用三个 snippet：

| Snippet | 效果 |
|---------|------|
| `vault-colors` | 文件浏览器中按类型着色 wiki 文件夹 |
| `ITS-Dataview-Cards` | 将 Dataview 表格转为视觉卡片网格 |
| `ITS-Image-Adjustments` | 精确控制图片尺寸（`|100`） |

---

## 系统要求

| 组件 | 最低版本 | 说明 |
|------|---------|------|
| Claude Code | latest | https://claude.com/claude-code |
| Obsidian | v1.9.10+（支持 Bases） | v1.6+ 可用 Dataview 回退 |
| Python | 3.10+ | 可选检索管线 + 测试套件 |
| Bash | 4.0+（或 zsh） | 安装脚本 |
| Git | any | vault 自动提交 |

**可选依赖:**
- **ollama** — `/wiki-retrieve` 的本地重排序
- **defuddle-cli** — `/defuddle` 的干净 web 提取
- **Anthropic API key** — `/wiki-retrieve` 的上下文前缀层（需 `--allow-egress` 同意）
- **Local REST API 插件** — REST API 方式 MCP

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

## 扩展：claude-canvas

如需更强大的可视层，可额外安装 [claude-canvas](https://github.com/AgriciDaniel/claude-canvas)：

```bash
claude plugin install AgriciDaniel/claude-canvas
```

提供 12 种模板、6 种布局算法、AI 图像生成、演示文稿和完整的画布编排功能，与 claude-obsidian vault 自动兼容。

---

## 相关链接

- [GitHub 仓库](https://github.com/AgriciDaniel/claude-obsidian)
- [深度解析博客](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)
- [YouTube 演示](https://www.youtube.com/watch?v=a2hgayvr-H4)
- [Karpathy LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [claude-canvas 扩展](https://github.com/AgriciDaniel/claude-canvas)
