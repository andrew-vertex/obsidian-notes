---
tags:
  - caveman
  - ai-skills
  - claude-code
  - codex
  - opencode
  - token-optimization
  - developer-tools
---
# caveman 安装、作用、使用与跨 Agent 集成（macOS）

下文整理 `JuliusBrussee/caveman` 的定位、能力、安装方式、使用方法，以及在 `Claude Code`、`Codex`、`OpenCode` 三类 agent 里的差异与实践建议。

截至 `2026-05-03` 核对，本文主要依据 `caveman` 官方 GitHub README、`hooks` 文档、`skills` 官方 README，以及 OpenCode 官方 `Rules` 文档整理。

官方地址：

- GitHub：<https://github.com/JuliusBrussee/caveman>
- README：<https://github.com/JuliusBrussee/caveman/blob/main/README.md>
- Hooks：<https://github.com/JuliusBrussee/caveman/blob/main/hooks/README.md>
- `skills` CLI：<https://github.com/vercel-labs/skills>
- OpenCode Rules：<https://opencode.ai/docs/rules>

## 目录

- [Key Takeaways](#key-takeaways)
- [caveman 是什么](#caveman-是什么)
- [它解决什么问题](#它解决什么问题)
- [macOS 安装前准备](#macos-安装前准备)
- [安装方式总览](#安装方式总览)
- [Claude Code：安装与使用](#claude-code安装与使用)
- [Codex：安装与使用](#codex安装与使用)
- [OpenCode：安装与使用](#opencode安装与使用)
- [三类 Agent 差异总表](#三类-agent-差异总表)
- [命令与触发方式速查](#命令与触发方式速查)
- [推荐安装路线](#推荐安装路线)
- [常见注意点](#常见注意点)
- [参考来源](#参考来源)

## Key Takeaways

- `caveman` 本质上是一个“输出压缩 skill / plugin”，不是模型本身，也不是新的 coding agent。
- 它主要压缩的是 `输出 tokens`，不是模型内部的 `reasoning / thinking tokens`。
- `README` 首页宣传值是“约 `75%` 输出 token 缩减”；同一份 README 的 benchmark 表格给出的平均值是 `65%`，区间约 `22%` 到 `87%`。这两个数字都要记住，别混成一个。
- `Claude Code` 支持最完整：插件安装、自动激活、状态栏 badge、`caveman-stats`、`cavecrew` 都更成熟。
- `Codex` 和 `OpenCode` 都能装，但形态更接近“skill 注入”而不是完整插件系统；自动常驻能力弱于 `Claude Code`。
- 如果你同时用多个 agent，macOS 上最省事的入口仍然是官方一键安装脚本；如果你只用单一 agent，再用该 agent 的原生安装方式会更清楚。

## caveman 是什么

| 维度 | 说明 |
| --- | --- |
| 项目 | `JuliusBrussee/caveman` |
| 定位 | 面向 AI coding agent 的超精简输出模式 |
| 核心目标 | 用更少字数保留同样技术信息，减少输出 token 成本与阅读负担 |
| 核心风格 | 去掉冠词、寒暄、铺垫、犹豫语，保留技术结论、命令、错误、代码与 API 名称 |
| 主要收益 | 更快读完、更少输出 token、更适合终端式协作 |
| 不做什么 | 不提升模型推理上限；不替代系统提示、项目规则或测试验证 |

## 它解决什么问题

| 痛点 | caveman 的处理方式 |
| --- | --- |
| 回答太长 | 把冗余礼貌语、过长背景和重复解释压掉 |
| token 成本高 | 优先减少最终输出长度 |
| 终端阅读负担大 | 让回答更像工程结论和行动项 |
| commit / review 太啰嗦 | 提供极简 `commit` 和一行 `review` 风格 |
| 会话上下文膨胀 | `caveman-compress` 可压缩 `CLAUDE.md` 等记忆文件 |

## macOS 安装前准备

| 项目 | 建议 | 说明 |
| --- | --- | --- |
| 系统 | `macOS` 开发机 | 本文默认 macOS |
| Git | 已安装 | 插件、repo、规则文件管理都更顺手 |
| `curl` | 已安装 | 一键脚本依赖 |
| Node.js / `npx` | 建议安装 | 从 `npx skills add ...` 命令形式可推断，`Codex` / `OpenCode` 路线通常需要它 |
| Agent 本体 | 先装好 | 先能正常运行 `Claude Code`、`Codex`、`OpenCode`，再装 caveman |

## 安装方式总览

### 1. 最省事：官方一键脚本

```bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash
```

| 特性 | 说明 |
| --- | --- |
| 适合谁 | 同时使用多个 agent，或不想逐个研究安装流程的人 |
| 脚本行为 | 检测本机已安装 agent，只对命中的 agent 执行安装 |
| 是否可重复执行 | 可以，README 明确说明可安全重跑 |
| macOS 价值 | 一次装好多种 agent 的 skill / plugin，省时间 |

### 2. 常用参数

| 参数 | 作用 | 适合场景 |
| --- | --- | --- |
| `--minimal` | 仅装 plugin / extension，不额外挂 hooks、MCP shrink、repo 规则 | 想先最小化试用 |
| `--all` | 装 plugin + hooks + statusline + MCP shrink + repo 规则 | 想一次到位 |
| `--only <agent>` | 只装指定 agent | 机器上装了很多 agent，但只想给一个启用 |
| `--with-init` | 往当前 repo 写规则文件或 `AGENTS.md` | 想要 repo 级 always-on |
| `--dry-run` | 只预览，不落地 | 想先看脚本会做什么 |
| `--list` | 打印支持的 agent 列表 | 想确认 `codex` / `opencode` 的 slug |

### 3. 手动按 Agent 安装

| Agent | 推荐命令 | 说明 |
| --- | --- | --- |
| `Claude Code` | `claude plugin marketplace add JuliusBrussee/caveman && claude plugin install caveman@caveman` | 官方 plugin 形态，能力最完整 |
| `Codex` | `npx skills add JuliusBrussee/caveman -a codex` | 当前 README 把 `Codex` 归入 `npx skills` 安装路径 |
| `OpenCode` | `npx skills add JuliusBrussee/caveman -a opencode` | 通过 `skills` 生态注入 |

说明：

- 截至 `2026-05-03`，`caveman` 仓库当前 `README` 的手动安装表里，`Codex` 与 `OpenCode` 都归在 `npx skills add ...` 这一路径。
- 如果你看到其他页面或旧截图写的是别的安装入口，优先以该仓库当前 `README` 为准。

## Claude Code：安装与使用

### Claude Code 为什么是最佳支持目标

| 项目 | 情况 |
| --- | --- |
| 安装形态 | 官方 plugin |
| 自动激活 | 支持 |
| 模式切换 | 支持 |
| 状态栏 badge | 支持 |
| `caveman-stats` | 支持 |
| `cavecrew` 子代理 | 支持 |
| 集成成熟度 | 三者里最高 |

### 安装

```bash
claude plugin marketplace add JuliusBrussee/caveman
claude plugin install caveman@caveman
```

如果你不想走 plugin，也可以只装 hooks：

```bash
bash <(curl -s https://raw.githubusercontent.com/JuliusBrussee/caveman/main/hooks/install.sh)
```

### 使用

| 操作 | 写法 |
| --- | --- |
| 开启默认 caveman | `/caveman` |
| 切到 lite | `/caveman lite` |
| 切到 ultra | `/caveman ultra` |
| 切到文言文 | `/caveman wenyan` |
| 停止 | `stop caveman` 或 `normal mode` |

### Claude Code 下你实际得到什么

| 能力 | 是否支持 | 说明 |
| --- | --- | --- |
| caveman mode | 支持 | 默认核心能力 |
| 自动激活 | 支持 | 通过 hook 注入会话上下文 |
| 状态栏 badge | 支持 | 可显示 `[CAVEMAN]`、`[CAVEMAN:ULTRA]` 等 |
| `caveman-commit` | 支持 | 极简 commit message |
| `caveman-review` | 支持 | 一行 code review |
| `caveman-help` | 支持 | 帮助卡 |
| `caveman-compress` | 支持 | 压缩 `CLAUDE.md` 等记忆文件 |
| `caveman-stats` | 支持 | 统计 token 节省情况 |
| `cavecrew-*` | 支持 | caveman 风格子代理 |

### 自动激活机制

根据 `hooks/README.md`，Claude Code 这条路线会用到：

| 组件                      | 作用                  |
| ----------------------- | ------------------- |
| `SessionStart` hook     | 会话开始时注入 caveman 规则  |
| `UserPromptSubmit` hook | 跟踪你切换到哪种 caveman 模式 |
| statusline script       | 在状态栏显示当前 caveman 状态 |

## Codex：安装与使用

### 安装

```bash
npx skills add JuliusBrussee/caveman -a codex
```

### 使用

| 操作 | 写法 | 说明 |
| --- | --- | --- |
| 开启默认 caveman | `$caveman` | `Codex` 用的是 `$caveman`，不是 `/caveman` |
| 切到 lite / full / ultra | 通过 skill 切换 | README 说明支持模式切换 |
| 停止 | `stop caveman` 或 `normal mode` | 用自然语言关闭 |

### Codex 下要特别记住的差异

| 项目 | 情况 |
| --- | --- |
| 命令前缀 | 用 `$caveman`，不是 `/caveman` |
| 自动激活 | 不是默认全局 always-on |
| repo 内自动启动 | `caveman` 仓库自己带了 `.codex/hooks.json`，所以在该仓库里跑 `Codex` 会自动启用 |
| 其他项目 | 需要手动触发，或自己复制 hook / 规则方案 |
| `caveman-commit` / `caveman-review` | README 明确说不在 Codex plugin bundle 内 |
| `caveman-compress` | README 表格显示可用 |

### Codex 适合什么用法

| 用法 | 建议 |
| --- | --- |
| 偶尔开 terse 模式 | 很合适，手动触发即可 |
| 想全局 always-on | 需要你额外补 repo 级 hook / 规则，不如 Claude Code 省心 |
| 主要想省输出 token | 仍然有价值 |

## OpenCode：安装与使用

### 安装

```bash
npx skills add JuliusBrussee/caveman -a opencode
```

### OpenCode 路线的本质

| 项目 | 情况 |
| --- | --- |
| 安装形态 | `skills` 生态注入 |
| 自动激活 | 默认不自动 |
| 最适合的增强方式 | 配合 `AGENTS.md` 做 global 或 project 级 always-on |
| 官方规则文件位置 | `~/.config/opencode/AGENTS.md` 或项目根 `AGENTS.md` |
| 强度切换 | 当前 README 没把 `OpenCode` 放进官方明确支持 `lite/full/ultra` 切换的分组 |

### 直接使用

| 操作 | 写法 |
| --- | --- |
| 当前会话临时开启 | `talk like caveman` |
| 另一种触发写法 | `caveman mode` |
| 更明确的要求 | `less tokens please` |
| 停止 | `stop caveman` 或 `normal mode` |

说明：

- 对 `OpenCode` 这类通过 `skills` 注入的 agent，更稳妥的理解是“它能识别对应 skill 指令”，而不是“它有一个官方内建 slash command”。
- 所以实操时，优先把它当成“自然语言触发的工作模式”更稳。
- 如果你非常在意 `lite / full / ultra / wenyan` 的细分切换，当前更稳的做法是把你要的默认风格直接写进 `AGENTS.md`，而不是预设 OpenCode 会像 `Claude Code` 那样原生暴露这些切换入口。

### 如果想在 OpenCode 里做 always-on

OpenCode 官方 `Rules` 文档说明：

| 规则类型 | 路径 |
| --- | --- |
| 全局规则 | `~/.config/opencode/AGENTS.md` |
| 项目规则 | `<project>/AGENTS.md` |

建议把下面这段放进全局或项目级 `AGENTS.md`：

```md
Terse like caveman. Technical substance exact. Only fluff die.
Drop filler, pleasantries, hedging, and redundant setup.
Fragments OK. Short synonyms OK. Keep code, commands, API names, and errors exact.
Stay in this mode until I say "stop caveman" or "normal mode".
```

### OpenCode 下你大致得到什么

| 能力 | 是否支持 | 说明 |
| --- | --- | --- |
| caveman mode | 支持 | 通过 `skills` 注入 |
| `caveman-commit` | 支持 | README 的 “Others” 分组包含此能力 |
| `caveman-review` | 支持 | 同上 |
| `caveman-help` | 支持 | 同上 |
| `caveman-compress` | 支持 | 同上 |
| `caveman-stats` | 不支持 | README 标注为 Claude Code only |
| 状态栏 badge | 不支持 | 仅 Claude Code 路线完整支持 |

## 三类 Agent 差异总表

| 维度 | Claude Code | Codex | OpenCode |
| --- | --- | --- | --- |
| 推荐安装方式 | plugin | `npx skills` | `npx skills` |
| 核心触发方式 | `/caveman` | `$caveman` | 自然语言触发为主 |
| 自动激活 | 强 | 中，需额外处理 | 弱，默认无 |
| 模式切换 | 支持 | 支持 | 当前 README 未明确列为 `OpenCode` 保证能力 |
| 状态栏 badge | 支持 | 不支持 | 不支持 |
| `caveman-stats` | 支持 | 不支持 | 不支持 |
| `caveman-commit` / `review` | 支持 | 不在 bundle 内 | 支持 |
| always-on 最佳手段 | plugin + hooks | hook / repo 级规则 | `AGENTS.md` |
| 整体成熟度 | 最高 | 中 | 中 |

## 命令与触发方式速查

| 目标 | Claude Code | Codex | OpenCode |
| --- | --- | --- | --- |
| 开启默认模式 | `/caveman` | `$caveman` | `talk like caveman` |
| 轻量模式 | `/caveman lite` | skill 切换 | 当前 README 未把它列为 `OpenCode` 明确承诺能力 |
| 默认 full | `/caveman full` | skill 切换 | 更适合在 `AGENTS.md` 里固定成默认风格 |
| 极简模式 | `/caveman ultra` | skill 切换 | 同上 |
| 文言文 | `/caveman wenyan` | 视 skill 支持情况触发 | 当前 README 未把它列为 `OpenCode` 明确承诺能力 |
| 停用 | `stop caveman` | `stop caveman` | `stop caveman` |

## 推荐安装路线

### 场景 1：你只用 Claude Code

| 建议 | 原因 |
| --- | --- |
| 直接装 plugin | 最完整、最省心 |
| 如果想更强集成，再跑官方脚本 | hooks、statusline、stats 都更自然 |

### 场景 2：你同时用 Claude Code、Codex、OpenCode

| 建议 | 原因 |
| --- | --- |
| 先确认三个 agent 本体都可用 | 避免脚本安装后找不到目标 agent |
| 再跑一键脚本 | 统一安装入口，后续维护成本最低 |
| 最后给 OpenCode 补 `AGENTS.md` | 解决它默认不 always-on 的问题 |

### 场景 3：你只想低风险试用

| 建议 | 原因 |
| --- | --- |
| 先用 `--minimal` | 不先动 hooks、statusline、MCP |
| 先只给一个 agent 装 | 更容易判断是否适合你 |

## 常见注意点

| 问题 | 说明 |
| --- | --- |
| 不要把 `75%` 和 `65%` 当成矛盾 | `75%` 是首页主宣传量级；README benchmark 表里的平均值是 `65%` |
| 不要把它当成“更聪明的模型” | 它主要是让输出更短，不是让 reasoning 更强 |
| `Codex` 命令别写错 | 当前文档里明确是 `$caveman`，不是 `/caveman` |
| `OpenCode` 默认不自动激活 | 需要自己补 `AGENTS.md` 才更像常驻模式 |
| `caveman-stats` 不要指望跨 agent 通用 | 当前是 Claude Code 特有能力 |
| 对多步骤高风险操作别过度压缩 | 原项目规则也强调：涉及安全警告、不可逆操作、多步骤歧义时，应该回到更清晰表达 |

## 参考来源

- `caveman` GitHub README：<https://github.com/JuliusBrussee/caveman/blob/main/README.md>
- `caveman` raw README：<https://raw.githubusercontent.com/JuliusBrussee/caveman/main/README.md>
- `caveman` hooks README：<https://github.com/JuliusBrussee/caveman/blob/main/hooks/README.md>
- `skills` 官方 README：<https://github.com/vercel-labs/skills>
- OpenCode Rules：<https://opencode.ai/docs/rules>
