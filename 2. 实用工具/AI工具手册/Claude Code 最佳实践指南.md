---
type: source
title: Claude Code 最佳实践指南
source_type: github
url: https://github.com/shanraisshan/Claude Code 最佳实践指南
author: Shayan (shanraisshan)
created: 2026-06-30
updated: 2026-06-30
tags:
  - source
  - claude-code
  - best-practice
  - agentic-engineering
  - subagents
  - skills
  - commands
  - workflows
---
# Claude Code 最佳实践指南

**GitHub**: [shanraisshan/Claude Code 最佳实践指南](https://github.com/shanraisshan/Claude Code 最佳实践指南)
**作者**: Shayan (shanraisshan)
**标语**: from vibe coding to agentic engineering — practice makes claude perfect
**Stars**: ~100k+ (曾登 GitHub Trending #1)
**支持**: [Disrupt.com](https://disrupt.com) · [ClaudeKit.cc](https://claudekit.cc)
**最后更新**: 2026-06-29

---

## 概述

这是一个 **Claude Code 最佳实践的权威参考仓库**，不是应用代码库，而是一个配置示范和知识集合。它全面覆盖了 Claude Code 的所有扩展机制（subagents、commands、skills、hooks、MCP、plugins、memory、settings），并通过一个完整的天气查询示例演示了 **Command → Agent → Skill** 的编排工作流模式。

项目的核心价值在于：
1. 系统化整理了 Claude Code 的所有配置能力和最佳实践
2. 收集了 Boris Cherny（Claude Code 创建者）和 Thariq（Anthropic）的 83 条实操技巧
3. 对比了 10 种主流开发工作流方法论
4. 提供了跨模型工作流方案（Claude Code + Codex/Gemini/GPT 等）
5. 汇总了 Skills、Agents 的社区集合资源

---

## 仓库结构

```
Claude Code 最佳实践指南/
├── README.md                          # 主文档：概念总览 + 工作流对比 + 资源汇总
├── CLAUDE.md                          # 本仓库自身的 Claude 配置（示范）
├── .claude/
│   ├── settings.json                  # 示范 settings 配置
│   ├── agents/                        # 示范 agent 定义
│   │   └── weather-agent.md
│   ├── commands/                      # 示范 command 定义
│   │   └── weather-orchestrator.md
│   ├── skills/                        # 示范 skill 定义
│   │   ├── weather-fetcher/SKILL.md
│   │   └── weather-svg-creator/SKILL.md
│   ├── hooks/                         # 跨平台声音通知 hooks 系统
│   └── rules/                         # 示范 rules 文件
├── best-practice/                     # 最佳实践文档（核心）
│   ├── claude-subagents.md            # Subagent 16 个 frontmatter 字段 + 5 种内置类型
│   ├── claude-skills.md               # Skill 16 个 frontmatter 字段 + 10 个内置 skills
│   ├── claude-commands.md             # Command 16 个 frontmatter 字段 + 85 个内置命令
│   ├── claude-memory.md               # CLAUDE.md 加载机制（祖先/后代）+ monorepo 最佳实践
│   ├── claude-mcp.md                  # MCP 服务器配置指南
│   ├── claude-settings.md             # Settings 配置参考
│   ├── claude-power-ups.md            # Power-ups 互动教程
│   └── claude-cli-startup-flags.md    # CLI 启动参数
├── implementation/                    # 实现示例
│   ├── claude-skills-implementation.md
│   ├── claude-subagents-implementation.md
│   ├── claude-commands-implementation.md
│   ├── claude-agent-teams-implementation.md
│   ├── claude-goal-implementation.md
│   └── claude-scheduled-tasks-implementation.md
├── tips/                              # Boris + Thariq 的 83 条技巧
│   ├── claude-boris-15-tips-30-mar-26.md
│   ├── claude-boris-13-tips-03-jan-26.md
│   ├── claude-boris-12-tips-12-feb-26.md
│   ├── claude-boris-10-tips-01-feb-26.md
│   ├── claude-boris-6-tips-16-apr-26.md
│   ├── claude-boris-2-tips-10-mar-26.md
│   ├── claude-boris-2-tips-25-mar-26.md
│   ├── claude-thariq-tips-17-mar-26.md
│   └── claude-thariq-tips-16-apr-26.md
├── reports/                           # 深度研究报告
│   ├── claude-agent-command-skill.md
│   ├── claude-agent-sdk-vs-cli-system-prompts.md
│   ├── why-harness-is-important.md
│   ├── llm-day-to-day-degradation.md
│   ├── claude-spinner-verbs-and-tips.md
│   ├── claude-global-vs-project-settings.md
│   ├── claude-advanced-tool-use.md
│   ├── learning-journey-weather-reporter-redesign.md
│   ├── claude-usage-and-rate-limits.md
│   ├── claude-skills-for-larger-mono-repos.md
│   ├── claude-agent-memory.md
│   └── claude-in-chrome-v-chrome-devtools-mcp.md
├── orchestration-workflow/            # 编排工作流演示
│   ├── orchestration-workflow.md
│   ├── orchestration-workflow.svg
│   ├── orchestration-workflow.gif
│   └── weather.svg
├── development-workflows/             # 开发工作流
│   ├── rpi/rpi-workflow.md
│   └── cross-model-workflow/cross-model-workflow.md
├── agent-teams/                       # Agent 团队配置
├── videos/                            # Claude Code 相关视频笔记
├── tutorial/day0/                     # Day 0 安装教程（Mac/Win/Linux）
└── !/                                 # 静态资源（badges, SVGs）
```

---

## 核心概念总览

README 以一个全面的概念表格开篇，列出了 Claude Code 的所有扩展点：

| 功能 | 位置 | 说明 |
|------|------|------|
| **Subagents** | `.claude/agents/<name>.md` | 子代理，16 个 frontmatter 字段，5 种内置类型 |
| **Commands** | `.claude/commands/<name>.md` | 自定义斜杠命令，16 个 frontmatter 字段，85 个内置命令 |
| **Skills** | `.claude/skills/<name>/SKILL.md` | 技能，16 个 frontmatter 字段，10 个内置 skills |
| **Workflows** | `.claude/workflows/` | 多代理编排工作流 |
| **Hooks** | `.claude/hooks/` | 生命周期钩子（18 种事件类型） |
| **MCP Servers** | `.mcp.json` / settings | Model Context Protocol 服务器 |
| **Plugins** | 可分发包 | 插件市场机制 |
| **Settings** | `.claude/settings.json` | 6 层配置层级 |
| **Status Line** | settings.json | 状态栏定制 |
| **Memory** | CLAUDE.md / `.claude/rules/` | 持久化记忆，祖先加载 + 后代懒加载 |
| **Checkpointing** | 自动（文件编辑追踪） | 支持 `/rewind` 回退 |
| **CLI Startup Flags** | `claude [flags]` | 命令行启动参数 |

### 🔥 热门新功能

README 也列出了 20+ 个 beta 阶段的热门功能：
- **Ultrareview**: 云端多代理代码审查
- **Ultraplan**: 云端计划模式
- **Auto Mode**: 免确认自动模式 (`Shift+Tab`)
- **Agent Teams**: 多代理团队协作
- **Advisor**: 双模型顾问策略
- **Computer Use**: 让 Claude 操控你的电脑
- **Agent SDK**: npm/pip 包，编程式使用
- **Ralph Wiggum Loop**: 自进化循环插件
- **Chrome**: 浏览器内运行
- **Scheduled Tasks / Routines**: 定时任务
- **Goal**: 跨轮次目标追踪
- **Git Worktrees**: 隔离工作树
- **Fast Mode**: 快速模式
- **No Flicker Mode**: 无闪烁全屏模式
- **Dynamic Workflows**: 动态多代理工作流
- **Agent View**: 后台代理监控
- **Voice Dictation**: 语音输入
- **Bundled Skills**: 内置技能（`/code-review`, `/batch`）
- **Devcontainers**: 开发容器支持
- **Channels**: 插件频道

---

## 编排工作流模式：Command → Agent → Skill

项目以一个完整的**天气查询系统**演示了核心编排模式：

### 架构流程

```
用户 → /weather-orchestrator (Command) 
         ├─ Step 1: 询问温度单位 (C/F)
         ├─ Step 2: 调用 weather-agent (Agent)
         │           └─ 预加载了 weather-fetcher skill
         │           └─ 从 Open-Meteo API 获取迪拜实时温度
         └─ Step 3: 调用 weather-svg-creator (Skill)
                     └─ 生成 SVG 天气卡片 + output.md
```

### 关键设计原则

1. **两种 Skill 模式并存**：
   - **Agent Skill（预加载）**: `weather-fetcher` 被预加载到 `weather-agent` 的上下文中，作为领域知识
   - **Skill（直接调用）**: `weather-svg-creator` 由 Command 通过 Skill 工具直接调用

2. **Command 作为编排器**: Command 负责用户交互和工作流协调
3. **Agent 负责数据获取**: Agent 使用预加载的 skill 获取数据后返回
4. **Skill 负责输出**: SVG 生成器独立运行，从对话上下文接收数据
5. **清晰的职责分离**: Fetch (Agent) → Render (Skill)，每个组件单一职责

---

## 开发工作流对比

项目整理了 **10 种主流开发工作流方法论** 的完整对比，它们都遵循 **Research → Plan → Execute → Review → Ship** 的共同模式：

| 工作流 | Stars | 命令数 | Agents | Skills | 特点 |
|--------|-------|--------|--------|--------|------|
| **Superpowers** | 241k | 0 | 0 | 14 | brainstorming → worktree → plan → subagent-dev → review → TDD → ship |
| **Everything Claude Code** | 223k | 84 | 67 | 272 | plan → tdd → code-review → security-scan → e2e → merge |
| **Matt Pocock Skills** | 150k | 0 | 0 | 35 | ask-matt → grill-with-docs → to-prd → prototype → tdd → diagnose → handoff |
| **gstack** | 118k | 0 | 0 | 53 | office-hours → plan-*-review → spec → design → qa → ship → retro |
| **Spec Kit** | 116k | 10 | 0 | 0 | constitution → specify → plan → tasks → implement → analyze → checklist |
| **Get Shit Done** | 65k | 67 | 33 | 0 | new-project → spec → plan → execute → review → validate → ship |
| **agent-skills** | 61k | 7 | 3 | 21 | spec → plan → build → test → review → ship |
| **OpenSpec** | 57k | 11 | 0 | 0 | explore → propose → new → apply → verify → sync → archive |
| **BMAD-METHOD** | 50k | 0 | 6 | 42 | forge-idea → product-brief → prd → architecture → epics → dev → review |
| **oh-my-claudecode** | 37k | 0 | 19 | 40 | team-plan → team-prd → team-exec → team-verify → team-fix |
| **Compound Engineering** | 22k | 1 | 0 | 26 | strategy → ideate → brainstorm → plan → work → review → compound |
| **HumanLayer** | 11k | 27 | 6 | 0 | research → plan → validate → implement → review → handoff → commit |

### 其他工作流
- **RPI Workflow** — 研究 → 计划 → 实施
- **Ralph Wiggum Loop** — 自进化循环
- **Andrej Karpathy Workflow** — OpenAI 创始成员的实践
- **Peter Steinberger Workflow** — OpenClaw 创建者的实践
- **Boris Cherny Workflow** — Claude Code 创建者的 83 条技巧
- **Thariq Workflow** — Anthropic 工程师的 session 管理技巧

---

## 跨模型工作流

项目还整理了在 Claude Code 中使用其他模型的方案：

### 三种桥接机制

1. **Plugin** — 另一个模型的 CLI 在 Claude Code 内部运行（如 `/codex:review`）
2. **MCP** — Claude Code 通过 MCP 将其他模型作为工具调用
3. **Router** — 将 Claude Code 的 API 端点切换到其他供应商

### 主要项目

| 项目 | Stars | 类型 | 桥接到 |
|------|-------|------|--------|
| **claude-code-router** | 34k | Router | OpenRouter, DeepSeek, Ollama, Gemini, Kimi, Qwen, Groq 等 |
| **CLIProxyAPI** | 32k | Router | Gemini CLI, Codex, Claude Code, Antigravity |
| **codex-plugin-cc** | 18k | Plugin | Codex / GPT-5（OpenAI 官方插件） |
| **pal-mcp-server** | 12k | MCP | 50+ 模型（Gemini, OpenAI, Azure, Grok, Ollama, OpenRouter） |

---

## Skills 和 Agents 集合资源

### Skill 集合（按 Stars 排序）

| 名称 | Stars | Skills 数 |
|------|-------|-----------|
| anthropics/skills | 156k | 17 |
| mattpocock/skills | 148k | 31 |
| Egonex-AI/Understand-Anything | 67k | 8 |
| wshobson/agents | 37k | 158 |
| scientific-agent-skills | 29k | 147 |
| impeccable | 27k | 1 (+7 设计领域) |
| agent-skills (addyosmani) | 27k | 21 |
| awesome-agent-skills | 26k | 1,497+ |
| claude-skills (alirezarezvani) | 15k | 246 |

### Agent 集合

| 名称 | Stars | Agents 数 |
|------|-------|-----------|
| agency-agents | 116k | 232 |
| awesome-claude-code-subagents | 22k | 156 |

---

## 关键最佳实践

### Subagent 定义（16 个 Frontmatter 字段）

关键字段：`name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `initialPrompt`, `color`

5 种内置 Agent 类型：
1. `general-purpose` — 通用多步任务
2. `Explore` — 只读代码搜索（haiku）
3. `Plan` — 计划模式研究（只读）
4. `statusline-setup` — 配置状态栏
5. `claude-code-guide` — Claude Code 功能问答

### Skill 定义（16 个 Frontmatter 字段）

关键字段：`name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell`

10 个内置 Skills：`code-review`, `batch`, `debug`, `loop`, `claude-api`, `fewer-permission-prompts`, `run`, `verify`, `run-skill-generator`, `simplify`

### Command 定义（16 个 Frontmatter 字段）

85 个内置 Commands，分类为：Auth(5), Config(16), Context(6), Debug(8), Export(2), Extensions(9), Memory(1), Model(6), Project(6), Remote(10), Session(15)

### Memory 机制

- **祖先加载**：向上遍历目录树加载所有 CLAUDE.md
- **后代懒加载**：只有在操作子目录文件时才加载对应的 CLAUDE.md
- **兄弟不加载**：同级目录的 CLAUDE.md 互不干扰
- 配置文件层级：Managed → CLI args → settings.local.json → settings.json → ~/.claude/settings.json

### 配置层级（6 层）

1. **Managed** (MDM/Plist/Registry) — 组织强制，不可覆盖
2. **命令行参数** — 单次会话覆盖
3. `.claude/settings.local.json` — 个人项目设置（git-ignored）
4. `.claude/settings.json` — 团队共享设置
5. `~/.claude/settings.json` — 全局个人默认
6. `hooks-config.local.json` 覆盖 `hooks-config.json`

---

## CLAUDE.md 编写要点（来自本项目自身）

本项目自身的 CLAUDE.md 也是一份示范，包含以下关键建议：

- CLAUDE.md 保持在 **200 行以内**，确保可靠遵守
- `.claude/rules/*.md` 带 `paths:` frontmatter 的会**懒加载**；不带 frontmatter 的会每次会话都加载
- 用 **commands** 来做工作流编排，而非独立 agents
- 创建**特定功能的 subagents + skills**（渐进式披露），而非通用 agents
- 在 ~50% 上下文使用时执行手动 `/compact`
- 复杂任务从 **plan mode** 开始
- 使用**人工门控的任务列表**工作流处理多步骤任务
- 将子任务拆到足够小，在 50% 上下文内完成
- Subagents **不能**通过 bash 命令调用其他 subagents，必须用 Agent 工具
- 每个文件单独提交（per-file commits），不要打包提交

---

## 相关资源

- 官方文档: https://code.claude.com/docs
- 官方 Skills 仓库: https://github.com/anthropics/skills
- Hooks 最佳实践: https://github.com/shanraisshan/claude-code-hooks
- Status Line 最佳实践: https://github.com/shanraisshan/claude-code-status-line
- Ralph Wiggum Loop: https://github.com/shanraisshan/ralph-wiggum-self-evolving-loop
- Draw JSON Architecture Skill: https://github.com/shanraisshan/draw-json-architecture-skill
- AI 术语表: https://github.com/shanraisshan/claude-code-codex-cursor-gemini
- Prompt Engineering 教程: https://github.com/anthropics/prompt-eng-interactive-tutorial
- Agent SDK 示例: https://github.com/anthropics/claude-agent-sdk-demos

---

## 适用人群

- **Claude Code 新手** — 通过 Day 0 教程和概念表格快速上手
- **高级用户** — 学习编排模式、工作流对比、跨模型方案
- **团队 Leader** — 参考配置层级、memory 策略、hooks 系统来规范化团队使用
- **插件/技能开发者** — 参考 frontmatter 字段规范和实现示例
- **技术决策者** — 通过 10 种工作流对比选择适合团队的方案
