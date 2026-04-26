---
tags:
  - claude-code
  - anthropic
  - cli
  - ai-coding
  - developer-tools
---
# Claude Code CLI 安装、配置、命令与最佳实践

面向终端开发场景，整理 Claude Code CLI 的安装方式、认证、配置体系、核心命令、复杂工程用法，以及可落地的个性化扩展方案。

截至 `2026-04-26` 核对，本文主要依据 Claude Code 官方文档整理。

官方地址：

- 概览：<https://code.claude.com/docs>
- 安装与升级：<https://code.claude.com/docs/en/getting-started>
- CLI Reference：<https://code.claude.com/docs/en/cli-reference>
- Commands：<https://code.claude.com/docs/en/commands>
- Settings：<https://code.claude.com/docs/en/configuration>
- Memory / CLAUDE.md：<https://code.claude.com/docs/en/memory>
- Permissions：<https://code.claude.com/docs/en/permissions>
- Hooks：<https://code.claude.com/docs/en/hooks>
- Hooks Guide：<https://code.claude.com/docs/en/hooks-guide>
- Skills：<https://code.claude.com/docs/en/slash-commands>
- Output Styles：<https://code.claude.com/docs/en/output-styles>
- Status Line：<https://code.claude.com/docs/en/statusline>
- Subagents：<https://code.claude.com/docs/en/sub-agents>
- Best Practices：<https://code.claude.com/docs/en/best-practices>
- Common Workflows：<https://code.claude.com/docs/en/common-workflows>
- Environment Variables：<https://code.claude.com/docs/en/env-vars>

## 目录

- [Key Takeaways](#key-takeaways)
- [Claude Code CLI 是什么](#claude-code-cli-是什么)
- [安装前先确认](#安装前先确认)
- [安装与升级](#安装与升级)
- [认证与计费模式](#认证与计费模式)
- [配置体系总览](#配置体系总览)
- [记忆与权限模式](#记忆与权限模式)
- [推荐目录结构](#推荐目录结构)
- [推荐基础配置](#推荐基础配置)
- [配置机制如何分工](#配置机制如何分工)
- [命令速查表](#命令速查表)
- [关键 CLI Flags 速查](#关键-cli-flags-速查)
- [关键环境变量](#关键环境变量)
- [复杂工程最佳实践](#复杂工程最佳实践)
- [个性化功能设计](#个性化功能设计)
- [常见误区](#常见误区)
- [参考来源](#参考来源)

## Key Takeaways

- Claude Code 现在官方更推荐 `native install`，也就是直接用安装脚本，而不是把 `npm install -g @anthropic-ai/claude-code` 当作首选。
- `Homebrew` 仍然是 macOS 上很好的团队分发方式，但它分成 `stable` 和 `latest` 两条 cask，且不会自动更新。
- Claude Code 真正好用的关键，不是“会不会安装”，而是是否把 `CLAUDE.md`、`.claude/settings.json`、`hooks`、`skills`、`subagents` 用对。
- 复杂工程里最重要的三件事是：控制上下文、让 Claude 能自验证、把规则做成可复用配置而不是每次临时提示。
- 个性化玩法并不需要官方单独支持；`statusLine`、`subagentStatusLine`、`hooks`、`skills` 已经足够做出“宠物、成就、专注模式、看板化提示”等扩展。

## Claude Code CLI 是什么

Claude Code 是 Anthropic 提供的 agentic coding CLI。它不是一个只会回答问题的聊天框，而是一个能：

- 读取代码库
- 搜索文件和内容
- 编辑文件
- 运行 shell 命令
- 使用子代理拆分任务
- 接入 MCP、插件、浏览器和 IDE
- 持久化会话与项目记忆

一句话理解：

- `Claude Code = 终端里的 AI 工程代理 + 配置系统 + 自动化扩展点`

## 安装前先确认

### 系统要求

根据官方安装文档，当前支持基线大致如下：

| 项目 | 要求 |
| --- | --- |
| macOS | `13.0+` |
| Windows | `Windows 10 1809+` 或 `Windows Server 2019+` |
| Linux | `Ubuntu 20.04+`、`Debian 10+`、`Alpine 3.19+` |
| CPU | `x64` 或 `ARM64` |
| 内存 | `4 GB+ RAM` |
| Shell | `Bash`、`Zsh`、`PowerShell`、`CMD` |
| 网络 | 需要联网，且所在地区需在 Anthropic 支持范围内 |

补充：

- 原生 Windows 需要 `Git for Windows`。
- `WSL 2` 是 Windows 下更适合工程开发的选择，尤其是你需要类 Unix 工具链和更好的沙箱行为时。
- 如果搜索功能异常，先确认 `ripgrep` 是否可用。

### 账号要求

Claude Code 不是免费计划默认可用的工具。官方当前支持的账号类型包括：

- Claude `Pro`
- Claude `Max`
- Claude `Team`
- Claude `Enterprise`
- Anthropic `Console`
- 第三方 provider：`Amazon Bedrock`、`Google Vertex AI`、`Microsoft Foundry`

## 安装与升级

### 安装方式选择建议

| 方式 | 推荐度 | 命令 | 适用场景 | 更新方式 |
| --- | --- | --- | --- | --- |
| Native Install | 最高 | `curl -fsSL https://claude.ai/install.sh \| bash` | macOS / Linux / WSL，想跟官方推荐保持一致 | 自动更新 + `claude update` |
| Homebrew Stable | 高 | `brew install --cask claude-code` | macOS 团队统一安装，偏稳 | `brew upgrade claude-code` |
| Homebrew Latest | 中 | `brew install --cask claude-code@latest` | 需要最快拿到新特性 | `brew upgrade claude-code@latest` |
| WinGet | 高 | `winget install Anthropic.ClaudeCode` | Windows 原生环境 | `winget upgrade Anthropic.ClaudeCode` |
| apt / dnf / apk | 高 | 见下方 | Linux 工作站、服务器、受管环境 | 系统包管理器 |
| npm | 中 | `npm install -g @anthropic-ai/claude-code` | 兼容旧习惯或 Node 工具链 | 遵循 npm 方式；官方首选仍是 native |

### 1. 推荐安装方式：Native Install

macOS / Linux / WSL：

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell：

```powershell
irm https://claude.ai/install.ps1 | iex
```

Windows CMD：

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

适合：

- 个人开发机
- 需要跟进最新能力
- 希望保留官方自动更新能力

### 2. macOS：Homebrew

Stable channel：

```bash
brew install --cask claude-code
```

Latest channel：

```bash
brew install --cask claude-code@latest
```

官方说明里，`claude-code` 通常比最新发布慢大约一周，更偏稳；`claude-code@latest` 会更快接收新版本。

### 3. Linux 包管理器

#### Debian / Ubuntu

```bash
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc \
  -o /etc/apt/keyrings/claude-code.asc
echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" \
  | sudo tee /etc/apt/sources.list.d/claude-code.list
sudo apt update
sudo apt install claude-code
```

升级：

```bash
sudo apt update && sudo apt upgrade claude-code
```

#### Fedora / RHEL

```bash
sudo tee /etc/yum.repos.d/claude-code.repo <<'EOF'
[claude-code]
name=Claude Code
baseurl=https://downloads.claude.ai/claude-code/rpm/stable
enabled=1
gpgcheck=1
gpgkey=https://downloads.claude.ai/keys/claude-code.asc
EOF

sudo dnf install claude-code
```

升级：

```bash
sudo dnf upgrade claude-code
```

#### Alpine

```bash
wget -O /etc/apk/keys/claude-code.rsa.pub \
  https://downloads.claude.ai/keys/claude-code.rsa.pub
echo "https://downloads.claude.ai/claude-code/apk/stable" >> /etc/apk/repositories
apk add claude-code
```

升级：

```bash
apk update && apk upgrade claude-code
```

Alpine 还要注意：

- 安装 `libgcc`、`libstdc++`、`ripgrep`
- 在 `settings.json` 中设置 `USE_BUILTIN_RIPGREP=0`

### 4. npm 安装

```bash
npm install -g @anthropic-ai/claude-code
```

注意：

- 官方明确不建议 `sudo npm install -g ...`
- 当前 npm 包本质上安装的是同一个 native binary，不是长期依赖 Node 运行 CLI
- 如果你只是正常开发使用，优先选 `native install` 或 `Homebrew`

### 5. 验证安装

```bash
claude --version
claude doctor
claude auth status --text
```

建议检查：

- 版本是否正常
- 安装类型是否健康
- 登录状态是否正常

### 6. 版本与更新策略

#### 立即更新

```bash
claude update
```

这个更适合 native install 路线。

#### 固定更新通道

```json
{
  "autoUpdatesChannel": "stable"
}
```

可选值：

- `"latest"`：最快拿到新特性
- `"stable"`：通常滞后一周左右，跳过明显有回归的版本

#### 设置最低版本地板

```json
{
  "autoUpdatesChannel": "stable",
  "minimumVersion": "2.1.100"
}
```

适合：

- 团队统一版本下限
- 避免从较新的 `latest` 退回太旧的 `stable`

### 7. 安全安装建议

如果你所在团队对供应链完整性敏感，建议校验官方签名。

官方发布签名指纹当前为：

```text
31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE
```

导入并查看指纹：

```bash
curl -fsSL https://downloads.claude.ai/keys/claude-code.asc | gpg --import
gpg --fingerprint security@anthropic.com
```

## 认证与计费模式

### 登录相关命令

| 命令 | 作用 |
| --- | --- |
| `claude auth login` | 登录 Anthropic 账号 |
| `claude auth login --console` | 使用 Anthropic Console 计费 |
| `claude auth logout` | 登出 |
| `claude auth status` | 输出 JSON 登录状态 |
| `claude auth status --text` | 输出易读文本状态 |
| `/login` | 交互会话里触发登录 |
| `/logout` | 交互会话里退出登录 |

### 认证模式建议

| 模式 | 适合谁 | 特点 |
| --- | --- | --- |
| Claude 订阅 | 个人开发者 | 上手最简单 |
| Console | 团队或 API 计费清晰化 | 成本归集更清楚 |
| Bedrock / Vertex / Foundry | 企业云治理场景 | 适合已有云合规体系 |

### `ANTHROPIC_API_KEY` 的注意事项

官方环境变量文档明确说明：

- 如果设置了 `ANTHROPIC_API_KEY`，它会优先于你已登录的 Claude 订阅
- `-p` 非交互模式下，只要这个变量存在，就会直接走 API Key

所以最佳实践是：

- 日常 CLI 主用订阅登录
- 脚本化、CI、网关代理、企业审计链路再切到 `ANTHROPIC_API_KEY`
- 不要在不知情的情况下同时保留登录态和 API Key，避免以为自己在走订阅，实际在烧 API 账单

## 配置体系总览

### 作用域

| 作用域 | 位置 | 影响范围 | 是否适合进版本库 |
| --- | --- | --- | --- |
| Managed | 系统级或服务器下发 | 全组织 / 全机器 | 由 IT 管理 |
| User | `~/.claude/` | 你在所有项目中的默认行为 | 否 |
| Project | 项目内 `.claude/` | 团队共享项目配置 | 是 |
| Local | `.claude/settings.local.json`、`CLAUDE.local.md` | 你在当前项目中的个性化偏好 | 否 |

### 优先级

从高到低：

1. Managed
2. CLI flags
3. Local
4. Project
5. User

这意味着：

- 团队项目配置会覆盖你的用户默认值
- 你临时传的 `--model`、`--permission-mode` 会覆盖文件配置

### 配置文件与用途

| 路径 | 用途 | 推荐内容 |
| --- | --- | --- |
| `~/.claude/settings.json` | 个人全局配置 | 语言、主题、输出风格、默认权限、用户级 hooks |
| `.claude/settings.json` | 团队共享项目配置 | 权限规则、团队 hooks、插件、项目约束 |
| `.claude/settings.local.json` | 个人项目覆盖 | 本机实验配置、临时 MCP、本地路径 |
| `CLAUDE.md` | 项目长期说明 | 构建命令、测试命令、架构规则、命名规范 |
| `CLAUDE.local.md` | 个人项目说明 | 本地沙箱地址、个人工作习惯、临时偏好 |
| `.claude/rules/*.md` | 规则模块化与按路径加载 | 前后端分离规则、测试规则、领域规则 |
| `.claude/skills/*/SKILL.md` | 可复用工作流与知识 | 发布流程、代码审查流程、域知识 |
| `.claude/agents/*.md` | 定制子代理 | reviewer、security-reviewer、migration-runner |
| `.mcp.json` | 项目 MCP 配置 | GitHub、Notion、Figma、DB 等 |
| `~/.claude.json` | Claude Code 内部状态 | OAuth、MCP、缓存、允许项等，通常不手改 |

## 记忆与权限模式

### `CLAUDE.md` 与 Auto Memory

Claude Code 当前有两套跨会话记忆机制：

| 机制 | 谁写入 | 内容类型 | 典型用途 |
| --- | --- | --- | --- |
| `CLAUDE.md` | 你 | 指令、规则、显式约束 | 构建命令、测试命令、项目架构规则 |
| Auto Memory | Claude | 它从你的纠正和会话里总结的 learnings | 常用命令、调试经验、你的偏好 |

理解方式：

- `CLAUDE.md` 更像“制度”
- Auto Memory 更像“经验”

最佳实践：

- 长期稳定规则写进 `CLAUDE.md`
- 反复出现但你懒得手记的偏好，让 Auto Memory 去累积
- 定期通过 `/memory` 审核 Auto Memory，避免把过时经验越记越多

### 权限模式

复杂工程里，权限模式最好是团队明确约定，而不是每个人凭感觉切换。

| 模式 | 行为 | 适合场景 |
| --- | --- | --- |
| `default` | 标准权限提示 | 日常开发默认值 |
| `acceptEdits` | 自动接受常见文件编辑和工作目录内文件系统操作 | 你信任本项目规则，但仍不想放开一切 |
| `plan` | 只读探索 | 需求分析、代码调研、制定计划 |
| `auto` | 用分类器审查命令与敏感写操作 | 需要更高自动化但仍保留保护栏 |
| `dontAsk` | 默认拒绝未明确允许的操作 | 极保守脚本化场景 |
| `bypassPermissions` | 跳过权限提示 | 只适合强沙箱、实验环境、完全受控 CI |

我的建议：

- 个人日常：`default` 或 `acceptEdits`
- 大任务前期：`plan`
- 无人值守批处理：`auto`
- 不要把 `bypassPermissions` 当作常规模式

## 推荐目录结构

复杂项目建议至少有这一层：

```text
your-project/
├── CLAUDE.md
├── CLAUDE.local.md
├── .mcp.json
└── .claude/
    ├── settings.json
    ├── settings.local.json
    ├── hooks/
    │   ├── format-and-test.sh
    │   └── protect-prod.sh
    ├── rules/
    │   ├── testing.md
    │   ├── backend/
    │   │   └── api-design.md
    │   └── frontend/
    │       └── ui-guidelines.md
    ├── agents/
    │   ├── reviewer.md
    │   └── security-reviewer.md
    ├── skills/
    │   ├── release-check/
    │   │   └── SKILL.md
    │   └── db-migration/
    │       └── SKILL.md
    └── output-styles/
        └── reviewer.md
```

## 推荐基础配置

### 1. 用户级 `~/.claude/settings.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "language": "chinese",
  "autoUpdatesChannel": "stable",
  "outputStyle": "Explanatory",
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Read(~/.ssh/**)"
    ]
  }
}
```

适合放这里的东西：

- 语言
- 更新通道
- 个人喜欢的输出风格
- 全局不该读的私密目录

### 2. 项目级 `.claude/settings.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/format-and-test.sh"
          }
        ]
      }
    ]
  }
}
```

适合放这里的东西：

- 敏感文件拒绝规则
- 团队必须执行的 hooks
- 项目插件
- 团队共享的模型 / 权限 / 工作流约束

### 3. 项目级 `CLAUDE.md`

```md
# Build And Test
- Install deps with `pnpm install`
- Run unit tests with `pnpm test --filter`
- Run lint with `pnpm lint`

# Code Conventions
- Use TypeScript strict mode
- Prefer existing abstractions in `src/shared/`
- Do not introduce new environment variables without updating `docs/config.md`

# Workflow
- For multi-file refactors, explore first, then make a plan, then implement
- Before finishing, run the narrowest relevant test command
```

写法建议：

- 短
- 可验证
- 只放“每次都重要”的规则

官方对 `CLAUDE.md` 的建议很明确：

- 目标控制在 `200` 行以内
- 过长会吃上下文，也会降低遵循率
- 如果它更像流程手册，就改成 `skills`
- 如果它只对某一类路径生效，就改成 `.claude/rules/`

## 配置机制如何分工

| 机制 | 解决什么问题 | 是否总是加载 | 最适合放什么 |
| --- | --- | --- | --- |
| `CLAUDE.md` | 持久指令 | 是 | 构建命令、测试命令、项目架构常识 |
| `.claude/rules/` | 模块化、按路径生效的规则 | 匹配时加载 | 前后端不同规范、目录级约束 |
| `hooks` | 强制自动动作 | 配置后自动触发 | 格式化、测试、拦截危险操作 |
| `skills` | 可复用工作流 / 知识 | 按需加载 | 发布流程、review checklist、领域知识 |
| `agents` | 专用子代理 | 调用时生效 | reviewer、安全审计、迁移执行器 |
| `output styles` | 改变回答风格 | 会话级 | Reviewer、Teacher、Architect 风格 |

一个很实用的判断规则：

- 想让 Claude “每次都知道” -> `CLAUDE.md`
- 想让某些规则“只在特定目录生效” -> `.claude/rules/`
- 想让某个动作“每次都执行，不能靠模型自觉” -> `hooks`
- 想把一套流程做成命令 -> `skills`
- 想把侧任务放进单独上下文 -> `agents`

## 命令速查表

### CLI 入口命令

| 命令 | 作用 | 典型用法 |
| --- | --- | --- |
| `claude` | 启动交互会话 | `claude` |
| `claude "query"` | 带首条 prompt 启动 | `claude "explain this repo"` |
| `claude -p "query"` | 非交互执行后退出 | `claude -p "explain this file"` |
| `cat file \| claude -p "query"` | 把管道内容交给 Claude | `cat logs.txt \| claude -p "summarize errors"` |
| `claude -c` | 继续当前目录最近会话 | `claude -c` |
| `claude -c -p "query"` | 接着上次上下文做非交互任务 | `claude -c -p "check for regressions"` |
| `claude -r "<session>" "query"` | 按会话 ID 或名称恢复 | `claude -r "oauth-migration" "continue"` |
| `claude update` | 立即更新 | `claude update` |
| `claude install [version]` | 安装或重装指定版本 / 通道 | `claude install stable` |
| `claude auth login` | 登录 | `claude auth login --console` |
| `claude auth logout` | 登出 | `claude auth logout` |
| `claude auth status` | 查看登录状态 | `claude auth status --text` |
| `claude agents` | 列出已配置子代理 | `claude agents` |
| `claude mcp` | 管理 MCP | `claude mcp` |
| `claude plugin` | 管理插件 | `claude plugin install xxx` |
| `claude remote-control` | 启动远程控制服务 | `claude remote-control --name "My Project"` |
| `claude setup-token` | 生成长生命周期 token | `claude setup-token` |

### 会话内命令总览

说明：

- 命令可见性受版本、平台、账号和计划影响
- 某些命令本质上是官方打包 skill，不是硬编码 built-in command
- 最准确的本机清单永远是 `/help`
- 某些文档或旧版本会把设置入口写成 `/config`；近期文档更常把它表述为 `/status` 打开的 Settings 界面，实际以你本机版本为准

### 1. 会话与上下文管理

| 命令 | 类型 | 作用 |
| --- | --- | --- |
| `/add-dir <path>` | Built-in | 给当前会话增加额外工作目录 |
| `/clear` | Built-in | 清空当前上下文，旧会话仍可恢复 |
| `/resume [session]` | Built-in | 恢复会话 |
| `/rename [name]` | Built-in | 给会话命名，便于后续恢复 |
| `/branch [name]` | Built-in | 从当前会话分叉出一条上下文分支 |
| `/rewind` | Built-in | 回退会话 / 代码状态或从某处开始总结 |
| `/btw <question>` | Built-in | 问一个不进入主上下文的临时问题 |
| `/plan [description]` | Built-in | 直接进入 plan mode |
| `/tasks` | Built-in | 查看和管理后台任务 |

### 2. 账号、状态、诊断

| 命令 | 类型 | 作用 |
| --- | --- | --- |
| `/help` | Built-in | 查看可用命令 |
| `/doctor` | Built-in | 诊断安装与配置问题 |
| `/status` | Built-in | 打开状态 / 设置界面，查看版本、模型、账号、连通性 |
| `/login` | Built-in | 登录 |
| `/logout` | Built-in | 登出 |
| `/release-notes` | Built-in | 查看变更日志 |
| `/insights` | Built-in | 生成使用分析报告 |

### 3. 配置、权限、记忆、扩展

| 命令 | 类型 | 作用 |
| --- | --- | --- |
| `/permissions` | Built-in | 管理 allow / ask / deny 规则 |
| `/sandbox` | Built-in | 开关沙箱 |
| `/hooks` | Built-in | 查看 hook 配置 |
| `/memory` | Built-in | 编辑 `CLAUDE.md`、查看 auto-memory |
| `/agents` | Built-in | 管理 agent 配置 |
| `/skills` | Built-in | 查看 skills |
| `/mcp` | Built-in | 管理 MCP 连接与 OAuth |
| `/plugin` | Built-in | 管理插件 |
| `/keybindings` | Built-in | 打开键位配置 |

### 4. 模型与响应风格

| 命令                           | 类型       | 作用                               |
| ---------------------------- | -------- | -------------------------------- |
| `/model [model]`             | Built-in | 选择使用的模型                          |
| `/effort [level\|auto]`      | Built-in | 设置推理深度（如 low、medium、high 或 auto） |
| `/fast [on\|off]`            | Built-in | 开启或关闭快速响应模式                      |
| `/theme`                     | Built-in | 切换界面主题                           |
| `/tui [default\|fullscreen]` | Built-in | 切换终端界面模式                         |
| `/terminal-setup`            | Built-in | 配置终端快捷键（例如 `Shift+Enter`）        |
| `/statusline`                | Built-in | 配置底部状态栏显示内容                      |
| `/voice [hold\|tap\|off]`    | Built-in | 配置语音输入模式                         |

### 5. 代码、评审、Git 与安全

| 命令                          | 类型       | 作用                             |
| --------------------------- | -------- | ------------------------------ |
| `/diff`                     | Built-in | 查看未提交变更和按轮次 diff               |
| `/review [PR]`              | Built-in | 做本地 PR / diff review           |
| `/security-review`          | Built-in | 对当前变更做安全审计                     |
| `/simplify [focus]`         | Skill    | 并行 review 并优化最近改动              |
| `/fewer-permission-prompts` | Skill    | 分析 transcripts，生成更精准 allowlist |

### 6. 自动化与协作

| 命令 | 类型 | 作用 |
| --- | --- | --- |
| `/loop [interval] [prompt]` | Skill | 周期性重复执行某个任务 |
| `/schedule [description]` | Built-in | 创建 / 更新 / 运行 routines |
| `/remote-control` | Built-in | 把当前 terminal 会话暴露给 web / app 控制 |
| `/remote-env` | Built-in | 配置远程 web session 的默认环境 |
| `/teleport` | Built-in | 把 web 会话拉回本地 terminal |
| `/install-github-app` | Built-in | 配置 Claude GitHub Actions app |
| `/web-setup` | Built-in | 连接 GitHub 凭据给 Claude Code on the web |
| `/team-onboarding` | Built-in | 根据你的使用历史生成团队 onboarding 指南 |

### 7. 其他平台 / 计划相关命令

| 命令 | 类型 | 作用 |
| --- | --- | --- |
| `/desktop` | Built-in | 把当前会话转到 Desktop |
| `/chrome` | Built-in | 配置 Chrome 集成 |
| `/setup-bedrock` | Built-in | 配置 Bedrock |
| `/setup-vertex` | Built-in | 配置 Vertex |
| `/mobile` | Built-in | 显示下载移动端的二维码 |
| `/upgrade` | Built-in | 升级订阅档位 |
| `/usage` | Built-in | 查看成本、使用量与限制 |
| `/powerup` | Built-in | 交互式功能教学 |

### 动态命令

| 类型 | 形式 | 说明 |
| --- | --- | --- |
| MCP Prompt | `/mcp__<server>__<prompt>` | 由 MCP server 动态暴露 |
| Skill 命令 | `/skill-name` | 由 `.claude/skills/*/SKILL.md` 生成 |
| 兼容旧命令 | `.claude/commands/*.md` | 仍兼容，但官方更推荐 skills |

## 关键 CLI Flags 速查

### 会话与目录

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--add-dir` | 追加工作目录 | `claude --add-dir ../shared ../infra` |
| `--name`, `-n` | 给会话命名 | `claude -n "auth-writer"` |
| `--continue`, `-c` | 继续最近会话 | `claude --continue` |
| `--resume`, `-r` | 恢复指定会话 | `claude --resume oauth-migration` |
| `--fork-session` | 恢复时另开 session ID | `claude --resume abc --fork-session` |
| `--session-id` | 强制指定 session UUID | `claude --session-id "<uuid>"` |
| `--worktree`, `-w` | 在隔离 git worktree 中启动 | `claude -w feature-auth` |
| `--tmux` | 为 worktree 创建 tmux 会话 | `claude -w feature-auth --tmux` |

### 权限与执行模式

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--permission-mode` | 指定权限模式 | `claude --permission-mode plan` |
| `--allowedTools` | 预允许工具 / 命令 | `claude --allowedTools "Bash(git diff *)" "Read"` |
| `--disallowedTools` | 彻底移除工具 | `claude --disallowedTools "Edit"` |
| `--tools` | 限制内置工具集合 | `claude --tools "Bash,Read,Edit"` |
| `--dangerously-skip-permissions` | 直接跳过权限提示 | `claude --dangerously-skip-permissions` |
| `--allow-dangerously-skip-permissions` | 允许后续切换到 bypass 模式 | `claude --permission-mode plan --allow-dangerously-skip-permissions` |

### 非交互与自动化

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--print`, `-p` | 非交互执行 | `claude -p "summarize repo"` |
| `--output-format` | 输出格式 `text/json/stream-json` | `claude -p --output-format json "list endpoints"` |
| `--input-format` | 输入格式 | `claude -p --input-format stream-json` |
| `--json-schema` | 结构化输出校验 | `claude -p --json-schema '{...}' "extract data"` |
| `--include-hook-events` | 把 hook 生命周期事件放入流输出 | `claude -p --output-format stream-json --include-hook-events` |
| `--include-partial-messages` | 输出 partial streaming events | `claude -p --output-format stream-json --include-partial-messages` |
| `--replay-user-messages` | 重新输出 stdin 中的 user messages | `claude -p --input-format stream-json --output-format stream-json --replay-user-messages` |
| `--max-turns` | 限制 agent turns | `claude -p --max-turns 5 "fix lint"` |
| `--max-budget-usd` | 限制花费 | `claude -p --max-budget-usd 3.00 "analyze logs"` |
| `--no-session-persistence` | 不写会话历史 | `claude -p --no-session-persistence "audit diff"` |

### 模型与系统提示

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--model` | 选择模型 | `claude --model sonnet` |
| `--effort` | 本次会话 effort | `claude --effort high` |
| `--fallback-model` | 打印模式下过载自动降级模型 | `claude -p --fallback-model sonnet "query"` |
| `--append-system-prompt` | 在默认系统提示后附加文本 | `claude --append-system-prompt "Always answer in Chinese"` |
| `--append-system-prompt-file` | 从文件附加系统提示 | `claude --append-system-prompt-file ./extra-rules.txt` |
| `--system-prompt` | 替换整套系统提示 | `claude --system-prompt "You are a reviewer"` |
| `--system-prompt-file` | 从文件替换系统提示 | `claude --system-prompt-file ./prompt.txt` |

### 调试、性能、极简模式

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--bare` | 跳过 hooks、skills、plugins、auto memory、CLAUDE.md 自动发现 | `claude --bare -p "query"` |
| `--debug` | 开 debug 日志 | `claude --debug "api,mcp"` |
| `--debug-file` | 把 debug 输出到指定文件 | `claude --debug-file /tmp/claude.log` |
| `--verbose` | 输出完整逐轮日志 | `claude --verbose` |
| `--disable-slash-commands` | 禁用 commands / skills | `claude --disable-slash-commands` |

### 远程、插件、IDE 与浏览器

| Flag | 作用 | 示例 |
| --- | --- | --- |
| `--remote` | 创建 web session | `claude --remote "Fix login bug"` |
| `--remote-control`, `--rc` | 启动可被 web / app 控制的会话 | `claude --remote-control "My Project"` |
| `--ide` | 如果可用，自动连接 IDE | `claude --ide` |
| `--chrome` | 启用 Chrome 集成 | `claude --chrome` |
| `--no-chrome` | 禁用 Chrome 集成 | `claude --no-chrome` |
| `--plugin-dir` | 临时加载插件目录 | `claude --plugin-dir ./my-plugins` |
| `--mcp-config` | 从文件或 JSON 加载 MCP 配置 | `claude --mcp-config ./mcp.json` |
| `--strict-mcp-config` | 只使用 `--mcp-config` 提供的 MCP | `claude --strict-mcp-config --mcp-config ./mcp.json` |

## 关键环境变量

完整列表请看官方环境变量文档，日常最常用的是下面这些：

| 变量 | 用途 | 建议 |
| --- | --- | --- |
| `ANTHROPIC_API_KEY` | 指定 API Key，优先于订阅登录 | 脚本 / CI / 网关场景用 |
| `ANTHROPIC_BASE_URL` | 指定 API 代理 / 网关地址 | 企业代理或中转层用 |
| `CLAUDE_CONFIG_DIR` | 改配置目录 | 多账号隔离很有用 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 设置 effort level | 团队可以统一默认值 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 让 `--add-dir` 也加载额外目录的 `CLAUDE.md` / rules | 多仓协作时很关键 |
| `DISABLE_AUTOUPDATER` | 禁掉后台自动更新检查 | 受管环境常用 |
| `CLAUDE_CODE_USE_BEDROCK` | 启用 Bedrock | 企业云 |
| `CLAUDE_CODE_USE_VERTEX` | 启用 Vertex | 企业云 |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Windows 原生使用 PowerShell tool | Windows 团队 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 关闭非必要流量 | 更严格的企业环境 |

一个很实用的别名做法：

```bash
alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work claude'
alias claude-personal='CLAUDE_CONFIG_DIR=~/.claude-personal claude'
```

适合：

- 工作 / 个人账号隔离
- 不同项目组之间彻底隔离状态与配置

## 复杂工程最佳实践

### 1. 用四阶段工作流，而不是一句话让 Claude 直接开干

复杂任务推荐：

1. Explore
2. Plan
3. Implement
4. Verify / Review

示例：

```text
1. 先只读 `src/auth` 和 `src/session`，理解现有登录和 token refresh 流程。
2. 给我一个实现 Google OAuth 的详细计划，不要改代码。
3. 按计划实现，补充测试，并运行最小必要测试集。
4. 再用 reviewer 子代理审查边界情况和安全问题。
```

什么时候不用 plan：

- 改一个文案
- 改一个变量名
- 一行日志

什么时候必须先 plan：

- 多文件重构
- 新功能跨前后端
- 需要迁移数据、配置、接口
- 你自己都不完全确定落点

### 2. 把验证能力交给 Claude，而不是只给任务

Claude 最容易出问题的地方，不是“不会改”，而是“改得像样但没验证”。

高杠杆做法：

- 在 `CLAUDE.md` 写清楚最小测试命令
- 给它能运行的 lint / test / build 命令
- 前端任务给可截图或可访问的页面
- 接口任务给 curl / fixture / e2e 命令

坏提示：

```text
修复登录 bug
```

好提示：

```text
修复登录 bug。问题是密码错误时页面空白。
改完后运行 `pnpm test auth --filter login`，
再验证错误提示文案是否仍是 `Invalid credentials`。
```

### 3. 单仓 / 多仓 / Monorepo 的配置策略要分层

推荐分法：

- 根目录 `CLAUDE.md`：全仓统一规则
- 子目录 `.claude/rules/` 或嵌套 `CLAUDE.md`：包级 / 服务级规则
- 用户级 `~/.claude/CLAUDE.md`：你的个人偏好，不污染仓库

如果你用 `--add-dir` 引入其他目录：

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

否则默认只给文件访问权，不会自动加载额外目录里的 `CLAUDE.md`、rules、local memory。

### 4. 大型改造优先用 `worktree`

对高风险任务，推荐：

```bash
claude -w feature-auth-refactor -n auth-writer
```

好处：

- 改动隔离
- 不污染主工作树
- 便于并行实验
- 适合一个 Writer 会话 + 一个 Reviewer 会话

如果你习惯 tmux：

```bash
claude -w feature-auth-refactor --tmux
```

### 5. 用双会话甚至多会话，而不是一个会话干到底

官方 best practices 很明确：多会话是复杂工程里的生产力倍增器。

推荐模式：

| 会话 | 角色 | 任务 |
| --- | --- | --- |
| Session A | Writer | 实现功能 |
| Session B | Reviewer | 独立 review diff、找边界条件 |
| Session C | Security Reviewer | 审计注入、权限、敏感数据 |

最常见的高价值模式：

- Writer / Reviewer
- Test Writer / Feature Writer
- Planner / Executor

一个可直接套用的模式：

```text
Session A:
Implement a rate limiter for our API endpoints.

Session B:
Review the rate limiter implementation. Focus on race conditions,
burst traffic, and consistency with existing middleware.
```

### 6. 用 subagents 做调查，不要让主上下文被搜索结果灌满

Claude 官方把这一点说得很直白：上下文是最贵的资源。

适合 subagent 的任务：

- “帮我查一下这个仓库里 OAuth 相关逻辑都在哪”
- “看看有没有现成的 retry 工具”
- “审查这次改动有没有安全问题”

示例提示：

```text
Use subagents to investigate how our authentication system handles token
refresh, and whether we already have reusable OAuth utilities.
```

### 7. 让规则自动执行：Hook 比“请记住”更可靠

适合做成 hook 的事情：

- 每次写文件后自动格式化
- 每次写文件后跑最小测试
- 禁止写生产配置目录
- 禁止对某些路径执行危险 Bash

例如，编辑后自动跑格式化 / 测试：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/format-and-test.sh"
          }
        ]
      }
    ]
  }
}
```

对安全更敏感的项目，建议再加：

- `PreToolUse` 拦截危险 Bash
- `SessionStart` 自动拉上下文信息
- `SessionEnd` 自动记录报告

### 8. 权限规则要把“建议”升级为“制度”

最基础的项目权限建议：

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  }
}
```

注意官方权限文档的一条关键提醒：

- `Read(./.env)` 只能拦 Claude 的内置读文件工具
- 不能阻止 `Bash(cat .env)` 这类 shell 访问

所以如果你真要做硬隔离：

- 用 sandbox
- 或用 `PreToolUse` hook 再拦一层

### 9. 非交互模式是复杂工程自动化的核心

最有用的几种组合：

#### JSON 输出

```bash
claude -p "List all API endpoints" --output-format json
```

#### Streaming JSON

```bash
claude -p "Analyze this log file" --output-format stream-json
```

#### 结构化输出

```bash
claude -p \
  --json-schema '{"type":"object","properties":{"risk":{"type":"string"}},"required":["risk"]}' \
  "Review this diff and output the top risk"
```

#### 成本 / 回合限制

```bash
claude -p --max-turns 4 --max-budget-usd 2.50 "triage these test failures"
```

适合：

- CI 检查
- 批量文件迁移
- 生成 structured report
- 定期扫描 TODO / 安全风险

### 10. 批处理任务要“限权 + 小步试跑”

官方推荐的 fan-out 模式很适合大规模迁移。

例如先列清单，再逐文件迁移：

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from API v1 to v2. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git diff *)"
done
```

最佳实践：

- 先挑 `2-3` 个文件试跑
- prompt 调通再批量
- `--allowedTools` 一定收紧
- 需要无人值守时优先 `--permission-mode auto`，不要默认 `bypassPermissions`

### 11. 善用 `CLAUDE.md` 的导入能力，但不要导太多

如果项目已有 `AGENTS.md`：

```md
@AGENTS.md

## Claude Code
- Use plan mode for billing changes.
```

如果你需要共享额外说明：

```md
See @README.md for project overview.
- Git workflow: @docs/git-workflow.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

但是要控制数量，因为 import 进来的内容一样会吃上下文。

### 12. Context discipline 比“高智商 prompt”更重要

复杂工程里最值钱的几个上下文习惯：

- 不相关任务之间立刻 `/clear`
- 简短侧问用 `/btw`
- 会话命名用 `/rename`
- 长任务分阶段恢复用 `claude --continue` / `--resume`
- 同一问题纠错超过两轮，就清上下文重开

这是官方 best practices 里反复强调的点。

## 个性化功能设计

下面这几项不是官方内置“玩具功能”，而是基于 Claude Code 官方扩展点做的可落地设计。它们的价值在于：

- 把抽象状态变成可视反馈
- 提升长期使用意愿
- 帮你控制上下文、成本、并发和质量

### 1. 终端宠物 `Claude Pet`

#### 设计目标

把状态栏做成一只“工程宠物”，根据当前会话状态变化表情、等级和台词。

#### 使用的官方扩展点

- `statusLine`
- `subagentStatusLine`
- `SessionEnd` hook
- `CLAUDE.md` 或 skill 命令

#### 数据来源

状态栏脚本能从 stdin 拿到 JSON，会包含：

- 当前模型
- `cwd`
- session id
- cost
- 持续时间
- context 使用比例
- 代码增删行数

#### 可视化规则示例

| 状态 | 触发条件 | 表现 |
| --- | --- | --- |
| 开心 | `used_percentage < 40%` 且测试通过 | `=^_^=` |
| 紧张 | `used_percentage >= 75%` | `=o_o=` 并提示 `/clear` 或 `/compact` |
| 生病 | hook 检测到测试失败 | `x_x` 并提示先修测试 |
| 升级 | 本次会话新增代码超过阈值且无失败 | `Lv.12 +1 XP` |
| 饥饿 | 超过 3 天没用 / 没有完成任务 | 显示 `feed me` |

#### 最小落地方案

`settings.json`：

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  },
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pet-session-end.sh"
          }
        ]
      }
    ]
  }
}
```

附加玩法：

- `/pet report`：做成 skill，输出宠物 XP、最近 7 天工作 streak、最常见任务类型
- `/pet feed`：实质是触发一次 cleanup / compact / status reset 工作流

### 2. 成就系统 `Claude Achievements`

把长期使用数据转成成就。

可统计：

- 连续 7 天有提交
- 连续 5 次任务都带验证
- 10 次 security review
- 20 次零失败 hook 执行
- 100 次 `/btw` 节省上下文

实现方式：

- `SessionEnd` hook 写入本地 `~/.claude/metrics.json`
- `statusLine` 展示当前 badge
- `/achievements` skill 汇总并生成周报

这个设计对复杂工程特别有价值，因为它会强化正确习惯，而不是只是“好玩”。

### 3. 专注模式 `Focus Operator`

做一个专门的 output style，把 Claude 变成更强硬的执行型工程代理。

适合：

- 重构日
- 故障排查
- 发布夜

策略：

- 输出更短
- 优先列风险
- 每次都要给验证步骤
- 上下文高时主动建议 `/clear` 或 `/compact`

这个更适合用 `output styles`，而不是 `CLAUDE.md`。因为它改变的是“回答风格”，不是项目规则。

### 4. 审核看板 `Reviewer HUD`

让 `subagentStatusLine` 显示每个 reviewer agent 的：

- 当前阶段
- token 消耗
- 审查重点
- 是否发现高风险问题

适合：

- 多子代理 review
- 并行迁移
- 大 PR 安全审查

你会很直观看到：

- 哪个 agent 卡住了
- 哪个 agent 成本异常高
- 哪个 agent 发现了 blocker

### 5. Boss Fight 模式

给高风险任务一个显式工作流：

- 切到 `red` 主题
- 启动 `security-reviewer`
- status line 显示 branch、context%、cost、danger level
- hook 禁止改生产目录和 secrets

适合：

- 支付
- 鉴权
- 生产脚本
- 数据迁移

## 常见误区

### 1. 还在把 npm 安装当作唯一正途

现在官方安装首选已经是 native install。npm 仍可用，但不应再默认认为它是标准路径。

### 2. `CLAUDE.md` 写成百科全书

太长会直接降低效果。它应该像“项目工作规则”，不是“系统设计文档全集”。

### 3. 一个会话里什么都聊

这是上下文污染最常见来源。任务切换就 `/clear`。

### 4. 让 Claude 写完就结束，不给验证手段

Claude 最需要的是可执行验证，而不是更长的赞美式提示词。

### 5. 在高风险环境默认开 `bypassPermissions`

只有非常受控的沙箱、CI、实验分支里才考虑。复杂工程默认优先：

- `plan`
- `default`
- `acceptEdits`
- `auto`

### 6. 以为权限 deny 规则能拦 Bash 的所有访问

官方文档明确说了，`Read(...)` / `Edit(...)` 规则主要作用于 Claude 内置文件工具，不等于操作系统级封锁。

### 7. 把所有流程都塞进 `CLAUDE.md`

正确做法通常是：

- 项目规则放 `CLAUDE.md`
- 目录或主题规则放 `.claude/rules/`
- 长流程放 `skills`
- 强制动作放 `hooks`

## 参考来源

- Claude Code Docs Overview：<https://code.claude.com/docs>
- Advanced Setup：<https://code.claude.com/docs/en/getting-started>
- CLI Reference：<https://code.claude.com/docs/en/cli-reference>
- Commands：<https://code.claude.com/docs/en/commands>
- Settings：<https://code.claude.com/docs/en/configuration>
- Memory / CLAUDE.md：<https://code.claude.com/docs/en/memory>
- Permissions：<https://code.claude.com/docs/en/permissions>
- Hooks：<https://code.claude.com/docs/en/hooks>
- Hooks Guide：<https://code.claude.com/docs/en/hooks-guide>
- Skills：<https://code.claude.com/docs/en/slash-commands>
- Output Styles：<https://code.claude.com/docs/en/output-styles>
- Status Line：<https://code.claude.com/docs/en/statusline>
- Subagents：<https://code.claude.com/docs/en/sub-agents>
- Best Practices：<https://code.claude.com/docs/en/best-practices>
- Common Workflows：<https://code.claude.com/docs/en/common-workflows>
- Environment Variables：<https://code.claude.com/docs/en/env-vars>
