---
title: "Claude CLI Tool 使用说明"
created: 2026-07-24
updated: 2026-07-24
tags:
  - tools
  - productivity
type: guide
status: distilled
---

# Claude CLI Tool 使用说明

## 目录

1. 文档目标
2. Claude CLI 是什么
3. 安装与版本
4. 登录与鉴权
5. 交互界面与基础操作
6. 输入 `/` 弹出的 Commands / Skills 菜单
7. 常用工作流
8. 配置、权限与沙箱
9. MCP / Plugins / Agents / Skills
10. 排障与注意事项
11. 当前本机环境补充说明
12. 参考链接

## 1. 文档目标

这份文档用于整理本机安装的 `Claude Code CLI` 使用方法，重点覆盖：

- 安装与升级
- 登录与运行模式
- 顶层 CLI 子命令
- 交互式 `/` 命令菜单
- 权限模式与安全边界
- MCP、插件、背景 agents、skills
- 常见排障

本文内容以以下证据为准：

- 本机 `claude` 二进制与 `--help`
- 本机安装包 `@anthropic-ai/claude-code`
- Claude Code 官方文档

## 2. Claude CLI 是什么

`Claude CLI`，也就是 `Claude Code` 的终端形态，是 Anthropic 提供的本地命令行编码代理工具。

它可以：

- 读取和理解代码仓库
- 修改文件
- 执行 shell 命令
- 通过 MCP 连接外部系统
- 使用 skills、plugins、hooks 扩展能力
- 在交互式会话里通过 `/` 命令切换模式、查看状态、运行工作流

和普通问答型 CLI 不同，`Claude Code` 更像一个“能读代码、改代码、跑命令、跨文件处理任务”的终端 agent。

## 3. 安装与版本

### 3.1 官方安装方式

Claude Code 官方文档当前给出的 CLI 安装方式包括：

- 原生安装脚本，macOS / Linux / WSL：

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

- Windows PowerShell：

```powershell
irm https://claude.ai/install.ps1 | iex
```

- Windows CMD：

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

- Homebrew：

```bash
brew install --cask claude-code
```

- WinGet：

```powershell
winget install Anthropic.ClaudeCode
```

- `npm`：

```bash
npm install -g @anthropic-ai/claude-code
```

### 3.2 自动更新与手动升级

官方文档说明：

- 原生安装脚本安装的版本会在后台自动更新
- Homebrew 和 WinGet 安装不会自动更新
- `claude update` / `claude upgrade` 是顶层升级命令

常见升级方式：

```bash
claude update
```

```bash
brew upgrade claude-code
```

```powershell
winget upgrade Anthropic.ClaudeCode
```

### 3.3 本机当前版本

当前环境确认到的版本是：

```bash
2.1.196 (Claude Code)
```

包管理信息：

- npm 包名：`@anthropic-ai/claude-code`
- 本机包版本：`2.1.196`

## 4. 登录与鉴权

### 4.1 Anthropic 账号登录

顶层命令：

```bash
claude auth login
```

这是最常规的登录方式，适合：

- 直接使用 Claude 订阅
- 使用官方账号能力
- 在桌面 / Web / CLI 之间协同会话

### 4.2 Console / API 计费登录

官方 CLI 参考里明确说明：

- `claude auth login --console`

适用于：

- 走 Anthropic Console 账单
- 偏 API / 团队计费场景

### 4.3 SSO 与邮箱预填

官方 CLI 参考说明：

- `claude auth login --sso`
  强制走 SSO
- `claude auth login --email <your@email>`
  预填邮箱

### 4.4 登录状态与登出

- 查看状态：

```bash
claude auth status
```

- 人类可读文本输出：

```bash
claude auth status --text
```

- 登出：

```bash
claude auth logout
```

### 4.5 长期 token

顶层命令里还有：

```bash
claude setup-token
```

官方文档说明它用于：

- 为 CI 或脚本生成长效 OAuth token
- 需要 Claude 订阅

## 5. 交互界面与基础操作

不带子命令启动时，`claude` 默认进入交互式会话：

```bash
claude
```

### 5.1 两类命令不要混淆

和 Codex 一样，这里有两层命令体系：

- 顶层 CLI 子命令
  例如 `claude auth`、`claude mcp`、`claude plugin`、`claude agents`
- 会话内 `/` 命令
  例如 `/model`、`/permissions`、`/compact`、`/review`

### 5.2 常见启动方式

- 在当前目录启动：

```bash
claude
```

- 启动时带初始 prompt：

```bash
claude "explain this project"
```

- 非交互输出：

```bash
claude -p "summarize the failing tests"
```

- 继续当前目录最近会话：

```bash
claude -c
```

- 恢复指定会话：

```bash
claude -r "auth-refactor"
```

- 启动时指定模型：

```bash
claude --model sonnet
```

- 启动时指定权限模式：

```bash
claude --permission-mode plan
```

### 5.3 常见顶层 CLI 子命令

根据本机 `claude --help` 与官方 CLI reference，常见顶层命令包括：

- `claude auth`
  管理登录状态
- `claude agents`
  管理背景 agents / parallel sessions
- `claude doctor`
  诊断安装和配置
- `claude mcp`
  管理 MCP 服务器
- `claude plugin` / `claude plugins`
  管理插件
- `claude project`
  管理项目状态和本地项目数据
- `claude install`
  安装或重装指定版本
- `claude update` / `claude upgrade`
  更新
- `claude ultrareview`
  非交互式云端多 agent code review

### 5.4 常见全局参数

本机 `claude --help` 里可确认的高频参数包括：

- `--model <model>`
- `--effort <level>`
- `--permission-mode <mode>`
- `--add-dir <directories...>`
- `--mcp-config <configs...>`
- `--plugin-dir <path>`
- `--settings <file-or-json>`
- `--agent <agent>`
- `--bg` / `--background`
- `--dangerously-skip-permissions`
- `--safe-mode`
- `--bare`

## 6. 输入 `/` 弹出的 Commands / Skills 菜单

这是 Claude CLI 交互模式里最重要的一部分。

### 6.1 Claude 的 `/` 菜单和 Codex 不完全一样

Claude 官方文档明确区分了三类条目：

- Built-in command
  逻辑直接写在 CLI 里
- Skill
  一种“提示式能力包”，本质是加载一段专门指令给 Claude
- Workflow
  一种带多 subagent / 并行 orchestration 的高级工作流

因此 Claude 的 `/` 菜单比 Codex 更像：

- 内置命令
- 官方 bundled skills
- 官方 workflows
- 用户自定义 skills
- MCP 暴露出的 prompts

### 6.2 官方文档确认的核心行为

官方 `Commands` 页面明确说明：

- 输入单独 `/` 可以列出你当前可用的全部 commands
- 输入 `/` 后继续键入可以过滤
- 只有出现在消息开头的 `/command` 才会被识别
- 文本跟在命令名后面，会作为参数

另外从 `v2.1.199` 起，skills 允许链式调用：

```text
/skill-a /skill-b do XYZ
```

最多可串 6 个 skill。

### 6.3 按场景整理的高频 `/` 命令

下面不是“全量逐字抄表”，而是把官方文档里最值得记住的命令先分组整理。

#### 会话与上下文

| 命令 | 作用 | 备注 |
|---|---|---|
| `/clear [name]` | 清空当前上下文并开始新对话 | 旧会话仍可 `/resume` |
| `/compact [instructions]` | 压缩历史，释放上下文 | 可附加总结重点 |
| `/context [all]` | 查看上下文占用 | `all` 展开更多明细 |
| `/resume [session]` | 恢复会话或打开 picker | 别名 `/continue` |
| `/branch [name]` | 从当前会话复制出分支会话 | 自己切过去继续 |
| `/rewind` | 回滚代码和/或会话到之前状态 | 别名 `/checkpoint`、`/undo` |
| `/rename [name]` | 重命名当前会话 | |
| `/export [filename]` | 导出当前对话文本 | |

#### 模型与执行模式

| 命令 | 作用 | 备注 |
|---|---|---|
| `/model [model]` | 切换模型 | 不带参数会打开 picker |
| `/effort [level\|auto]` | 调整推理强度 | 支持 `low` 到 `max`、`ultracode` |
| `/plan [description]` | 进入 plan mode | 可直接附任务描述 |
| `/permissions` | 管理 allow / ask / deny 规则 | 别名 `/allowed-tools` |
| `/sandbox` | 切换 sandbox mode | 仅支持的平台可见 |
| `/fast [on\|off]` | 切换 fast mode | |
| `/config [key=value ...]` | 打开或直接修改设置 | 别名 `/settings` |
| `/status` | 打开状态页 | 看版本、模型、账号、连接状态 |

#### 检查、评审与调试

| 命令 | 作用 | 备注 |
|---|---|---|
| `/diff` | 打开 diff viewer | 看未提交变更与逐 turn diff |
| `/review [PR]` | 快速单轮只读评审 GitHub PR | |
| `/code-review [level] [--fix] [--comment] [target]` | 更完整的 diff review | 属于 Skill |
| `/security-review` | 安全审查当前变更 | |
| `/debug [description]` | 诊断运行时问题并启用调试日志 | Skill |
| `/doctor` | 检查安装、PATH、配置、技能、插件等问题 | Skill，别名 `/checkup` |
| `/feedback [report]` | 提交反馈或 bug | 别名 `/bug`、`/share` |

#### 并行与 agents

| 命令 | 作用 | 备注 |
|---|---|---|
| `/fork <directive>` | 派一个 forked subagent 去做旁支任务 | 和 `/branch` 不同 |
| `/tasks` | 查看当前 session 背景任务 | 文档中也提到 `/bashes` |
| `/background [prompt]` | 把整个 session 挂到后台继续跑 | 别名 `/bg` |
| `/goal [condition\|clear]` | 设定长期目标 | Claude 会跨 turn 持续推进 |
| `/agents` | 当前版本主要提示你通过 Claude 或 `.claude/agents/` 管理 subagents | 旧版本行为不同 |

#### 外部能力与生态

| 命令 | 作用 | 备注 |
|---|---|---|
| `/mcp [reconnect ...\|enable\|disable ...]` | 管理 MCP 连接与认证 | 无参数时打开交互列表 |
| `/plugin [subcommand]` | 管理 plugins | |
| `/hooks` | 查看 hooks 配置 | |
| `/skills` | 列出可用 skills | 支持排序与可见性控制 |
| `/memory` | 管理 `CLAUDE.md` memory 和 auto-memory | |
| `/claude-api [...]` | 加载 Claude API 相关技能 | Skill |

#### 平台与辅助能力

| 命令 | 作用 | 备注 |
|---|---|---|
| `/desktop` | 把当前 session 交给桌面端继续 | 别名 `/app` |
| `/remote-control` | 让当前 session 可被远程控制 | 别名 `/rc` |
| `/teleport` | 把 Claude Web 会话拉回终端 | 别名 `/tp` |
| `/ide` | 管理 IDE 集成 | |
| `/theme` | 修改主题 | |
| `/keybindings` | 打开快捷键配置文件 | |
| `/terminal-setup` | 配置终端快捷键行为 | |
| `/voice [hold\|tap\|off]` | 控制语音输入 | |

### 6.4 官方确认存在的 bundled skills

官方 `Skills` 文档明确指出，Claude Code 自带一批 bundled skills，默认每个 session 都可用，除非被显式关闭。文档中点名的包括：

- `/doctor`
- `/code-review`
- `/batch`
- `/debug`
- `/loop`
- `/claude-api`

### 6.5 你本机实际还会受哪些东西影响

除了官方内置 commands / bundled skills，本机还会额外受到这些内容影响：

- `~/.claude/CLAUDE.md`
  全局自定义规则
- `~/.claude/skills/`
  个人全局 skills
- 当前项目里的 `.claude/skills/`
  项目级 skills
- 插件带来的 skills / commands / hooks / MCP
- MCP 暴露的 prompts

也就是说，你机器里看到的 `/` 菜单，未必和别人完全相同。

## 7. 常用工作流

### 7.1 第一次进入一个新仓库

建议顺序：

1. 进入项目目录
2. 运行：

```bash
claude
```

3. 先确认几个入口：

- `/init`
- `/memory`
- `/permissions`
- `/model`
- `/mcp`

4. 如果项目还没有说明文件，先用 `/init` 生成 `CLAUDE.md`
5. 如果是大仓库，先用 `/plan` 再开始改

### 7.2 日常开发

高频循环通常是：

1. 用自然语言描述任务
2. 需要规划时用 `/plan`
3. 需要切模型时用 `/model`
4. 需要看上下文时用 `/context`
5. 需要清理上下文时用 `/compact`
6. 需要查看当前变更时用 `/diff`

### 7.3 长任务推进

Claude 的长任务支持比很多 CLI 更完整，推荐组合：

- `/goal`
- `/plan`
- `/fork`
- `/tasks`
- `/background`

典型用法：

- 先 `/goal` 设目标
- 用 `/plan` 做方案
- 让 `/fork` 或 `/background` 承担旁支任务

### 7.4 Review 工作流

常用分层：

- `/review`
  更快、更轻的单轮 review
- `/code-review`
  更系统的 diff review
- `/security-review`
  安全专项 review
- `claude ultrareview`
  顶层命令，云端多 agent 深度审查

### 7.5 会话管理工作流

- `/clear`
  开新上下文但保留历史会话可恢复
- `/resume`
  找回之前的上下文
- `/branch`
  自己切入另一个分支思路
- `/fork`
  把分支任务交给背景 subagent
- `/teleport`
  从 Web 把会话带回终端

## 8. 配置、权限与沙箱

### 8.1 权限模式总览

Claude 官方 `Permission modes` 文档当前给出 6 个模式：

- `default`
  只读，适合敏感操作或初次进入仓库
- `acceptEdits`
  自动接受文件修改与部分常见文件系统命令
- `plan`
  只探索和规划，不直接改代码
- `auto`
  少量提示、后台安全分类器审查
- `dontAsk`
  只允许预批准工具
- `bypassPermissions`
  跳过权限检查，风险最高

CLI 帮助里的 `--permission-mode` 也能确认这些模式：

- `acceptEdits`
- `auto`
- `bypassPermissions`
- `default`
- `dontAsk`
- `plan`

### 8.2 切换权限模式

官方文档说明：

- 在 CLI 里可用 `Shift+Tab` 循环切换常见模式
- 启动时可直接指定：

```bash
claude --permission-mode plan
```

- 也可以通过设置文件持久化默认值

### 8.3 `acceptEdits` 模式

适合：

- 你希望 Claude 少打断你
- 愿意在改完后统一 review diff

官方文档明确说它会自动放行：

- 文件编辑
- 一部分常见文件系统命令，例如 `mkdir`、`touch`、`mv`、`cp`

但仍然受这些边界约束：

- 工作目录范围
- `additionalDirectories`
- protected paths
- 其他非白名单 Bash 命令仍会提示

### 8.4 `plan` 模式

适合：

- 大改动前的探索和方案设计
- 先确认 Claude 的理解与实施路径

它会：

- 读文件
- 跑探索命令
- 生成计划
- 不直接改源码

还可以用单条命令直接进入：

```text
/plan fix the auth bug
```

### 8.5 `auto` 模式

`auto` 模式是 Claude 较强的自治模式，但官方文档强调它不是“无需审查”的安全替代品。

官方文档列出的关键信息：

- 由独立分类器审查动作
- 会阻断明显越权、危险、外部敏感目标操作
- 某些账户、计划、模型、provider 组合下才可用

官方列举的默认阻断类别包括：

- `curl | bash`
- 发送敏感数据到外部端点
- 生产部署与迁移
- 强制 push
- `git reset --hard`
- `terraform destroy`
- 输出 live credential / token

### 8.6 高风险选项

CLI 帮助明确给出两个需要特别小心的入口：

- `--dangerously-skip-permissions`
- `--allow-dangerously-skip-permissions`

使用建议：

- 本地日常开发尽量不要默认开启
- 只在隔离良好的 VM / sandbox / CI 容器中考虑

### 8.7 `safe-mode` 与 `bare`

这两个启动参数很有用，但用途不同：

- `--safe-mode`
  为了排查配置问题，禁用大部分自定义内容
- `--bare`
  极简模式，跳过 hooks、skills、plugins、MCP、auto-memory、CLAUDE.md 自动发现等

适用场景：

- 怀疑是插件 / skills / hooks 搞坏行为时，用 `--safe-mode`
- 想要最小开销、最少自动加载时，用 `--bare`

## 9. MCP / Plugins / Agents / Skills

### 9.1 MCP

Claude 对 MCP 的集成很深，既有顶层命令，也有会话内入口。

顶层：

```bash
claude mcp --help
```

本机确认到的子命令包括：

- `add`
- `add-from-claude-desktop`
- `add-json`
- `get`
- `list`
- `login`
- `logout`
- `remove`
- `reset-project-choices`
- `serve`

会话内：

- `/mcp`

官方文档说明它可以：

- 打开交互式 MCP 列表
- reconnect 某个 server
- enable / disable 某个 server 或全部 server

### 9.2 Plugins

顶层：

```bash
claude plugin --help
```

本机确认到的子命令包括：

- `details`
- `disable`
- `enable`
- `init` / `new`
- `install` / `i`
- `list`
- `marketplace`
- `prune` / `autoremove`
- `tag`
- `uninstall` / `remove`
- `update`
- `validate`

会话内：

- `/plugin`
- `/reload-plugins`

### 9.3 Agents 与背景会话

顶层：

```bash
claude agents
```

用于：

- 查看活动 background sessions
- 以 JSON 形式导出
- 为 dispatched sessions 指定 `model`、`effort`、`permission-mode`、`agent`

官方 CLI reference 还列出了一组相关命令：

- `claude attach <id>`
- `claude logs <id>`
- `claude stop <id>`
- `claude rm <id>`
- `claude respawn <id>`

### 9.4 Skills

Claude 的 skills 是它和 Codex 差异非常大的一块。

官方文档给出的关键点：

- skill 的入口就是 `/skill-name`
- skill 本质是懒加载的指令包
- skill 可以自动被 Claude 判断为相关而调用
- 你也可以显式输入 `/skill-name`

官方文档说明 skills 的存放位置：

- 个人：
  `~/.claude/skills/<skill-name>/SKILL.md`
- 项目：
  `.claude/skills/<skill-name>/SKILL.md`
- 插件：
  `<plugin>/skills/<skill-name>/SKILL.md`

并且旧的 `.claude/commands/` 仍然兼容，但官方推荐迁移到 skills。

### 9.5 何时用哪一个

- 想管理外部工具接入：MCP
- 想安装扩展包：Plugins
- 想并行跑长期任务：Agents / background sessions
- 想沉淀可重复 prompt/流程：Skills

## 10. 排障与注意事项

### 10.1 配置异常或行为怪异

优先尝试：

```bash
claude doctor
```

如果是会话内排查：

- `/doctor`

如果怀疑是配置或插件导致：

```bash
claude --safe-mode
```

或者：

```bash
claude --bare
```

### 10.2 `/` 菜单内容和别人不一样

这在 Claude 里非常正常，常见原因：

- 平台不同
- 版本不同
- 订阅计划不同
- 是否登录 Claude subscription
- 是否启用了某些 provider / integrations
- 是否加载了 skills / plugins / MCP prompts

### 10.3 恢复不到会话

官方 sessions 文档确认：

- session 按项目目录存储
- `claude --resume <session-id>` 的 session ID 查找范围受当前项目目录约束
- 通过名字恢复时，会跨当前仓库及其 worktrees 查找

所以常见排查方式：

1. 回到原项目目录再试
2. 用 `claude --resume` 不带参数打开 picker
3. 尽量给重要会话命名

### 10.4 权限模式误用

最容易踩坑的是：

- 长期在 `bypassPermissions`
- 没分清 `acceptEdits` 和 `plan`
- 在敏感仓库里直接启 `auto`

建议：

- 不确定时先用 `default` 或 `plan`
- 大改动前先 `/plan`
- 改完一定 `/diff` 自己再看一遍

### 10.5 自定义内容影响行为

Claude 受这些内容影响很大：

- `CLAUDE.md`
- `~/.claude/CLAUDE.md`
- `.claude/skills/`
- `~/.claude/skills/`
- plugins
- hooks
- MCP

所以“同一个版本，两个机器行为不同”是常态。

### 10.6 版本不一致

排查时先确认：

```bash
which claude
claude -v
```

以及 npm 包版本：

```bash
npm list -g @anthropic-ai/claude-code
```

## 11. 当前本机环境补充说明

这部分只记录你这台机器上已经确认到的特殊情况。

### 11.1 本机安装路径

当前 `claude` 来自：

```bash
/Users/yuanjianwei/.nvm/versions/node/v22.22.0/bin/claude
```

安装包目录：

```bash
/Users/yuanjianwei/.nvm/versions/node/v22.22.0/lib/node_modules/@anthropic-ai/claude-code
```

### 11.2 本机存在全局 `~/.claude/CLAUDE.md`

当前检测到：

- `~/.claude/CLAUDE.md` 存在

这意味着你每次运行 Claude 时，都可能受到全局自定义规则影响。

### 11.3 本机 `~/.claude/skills/` 当前未发现可直接列出的 `SKILL.md`

本次检查没有在：

- `~/.claude/skills/*/SKILL.md`

下读到现成文件列表输出。

这不代表没有 skill 来源，只代表本次确认中没有直接从这里列出个人 skill 文件。你的 `/` 菜单仍可能受以下来源影响：

- bundled skills
- 项目内 `.claude/skills/`
- plugins
- MCP prompts

## 12. 参考链接

- 本机包 README：
  `/Users/yuanjianwei/.nvm/versions/node/v22.22.0/lib/node_modules/@anthropic-ai/claude-code/README.md`
- Claude Code 概览：
  https://code.claude.com/docs/en/overview
- CLI 参考：
  https://code.claude.com/docs/en/cli-reference
- Commands 参考：
  https://code.claude.com/docs/en/commands
- Permission modes：
  https://code.claude.com/docs/en/permission-modes
- Sessions：
  https://code.claude.com/docs/en/sessions
- Skills：
  https://code.claude.com/docs/en/skills
- Hooks：
  https://code.claude.com/docs/en/hooks
