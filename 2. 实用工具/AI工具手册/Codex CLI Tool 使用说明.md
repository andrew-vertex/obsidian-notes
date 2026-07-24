---
title: "Codex CLI Tool 使用说明"
created: 2026-07-24
updated: 2026-07-24
tags:
  - tools
  - productivity
type: guide
status: distilled
---

# Codex CLI Tool 使用说明

## 目录

1. 文档目标
2. Codex CLI 是什么
3. 安装与版本
4. 登录与鉴权
5. 交互界面与基础操作
6. Slash Commands：输入 `/` 弹出的命令菜单
7. 常用工作流
8. 配置、权限与沙箱
9. MCP / Apps / Plugins
10. 排障与注意事项
11. 参考链接

## 1. 文档目标

这份文档用于沉淀 `Codex CLI` 的使用方法，优先覆盖日常高频操作、交互命令、权限模型和排障方法。

## 2. Codex CLI 是什么

`Codex CLI` 是 OpenAI 提供的本地命令行编码代理工具。它支持：

- 在终端里与模型交互
- 读取和修改工作区文件
- 执行 shell 命令
- 在受控权限下访问网络、工具、MCP 服务
- 通过 `/` 命令快速切换模式、查看状态、打开配置入口

## 3. 安装与版本

官方 README 目前给出的安装方式有 4 类：

### 3.1 官方安装方式

- macOS / Linux 官方脚本

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

- Windows 官方脚本

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

- `npm` 安装

```bash
npm install -g @openai/codex
```

- Homebrew 安装

```bash
brew install --cask codex
```

### 3.2 其他获取方式

也可以从 GitHub Release 直接下载平台二进制。官方 README 给出的常见文件包括：

- macOS Apple Silicon：`codex-aarch64-apple-darwin.tar.gz`
- macOS x86_64：`codex-x86_64-apple-darwin.tar.gz`
- Linux x86_64：`codex-x86_64-unknown-linux-musl.tar.gz`
- Linux arm64：`codex-aarch64-unknown-linux-musl.tar.gz`

解压后一般需要把产物重命名为 `codex` 并加入 `PATH`。

### 3.3 当前本机版本

当前环境确认到的版本是：

```bash
codex-cli 0.144.1
```

### 3.4 升级方式

- 如果通过包管理器安装，优先使用对应包管理器升级
- `codex update` 是官方提供的顶层子命令之一
- 如果安装源混用，先确认 `which codex` 指向的实际路径

## 4. 登录与鉴权

`Codex CLI` 当前有 3 类主要登录方式。

### 4.1 ChatGPT 账号登录

这是官方 README 推荐的默认方式。启动 `codex` 后，欢迎界面可直接选择：

- `Sign in with ChatGPT`

适用场景：

- 希望直接使用 ChatGPT 订阅计划中的 Codex 能力
- 不想单独维护 API Key

官方 README 明确提到，ChatGPT 登录适用于：

- Plus
- Pro
- Business
- Edu
- Enterprise

### 4.2 Device Code 登录

欢迎界面里还会提供：

- `Sign in with Device Code`

适用场景：

- 当前机器不方便直接拉起浏览器
- 远程终端或受限桌面环境

### 4.3 API Key / Access Token 登录

CLI 帮助里可以确认 `codex login` 支持：

- `--with-api-key`
- `--with-access-token`
- `--device-auth`

示例：

```bash
printenv OPENAI_API_KEY | codex login --with-api-key
```

```bash
printenv CODEX_ACCESS_TOKEN | codex login --with-access-token
```

### 4.4 登录状态检查与登出

- 查看登录状态：

```bash
codex login status
```

- 登出：

```bash
codex logout
```

### 4.5 登录方式差异

- ChatGPT 登录更适合交互式个人使用
- API Key 更适合自动化、隔离环境、可控计费
- 某些功能依赖 ChatGPT 身份，例如 `/usage` 在未使用 ChatGPT 登录时会被限制

## 5. 交互界面与基础操作

`Codex CLI` 在不带子命令启动时，会进入交互式 TUI。输入单独 `/` 会弹出内置命令菜单，这个菜单不是顶层 shell 子命令，而是 TUI 内的 `slash commands`。

### 5.1 两类命令不要混淆

需要注意：

- 命令是否显示，受平台、登录状态、feature flag、当前会话状态影响
- 某些命令支持内联参数
- 某些命令在 side conversation 或任务运行中不可用

对比：

- 顶层 CLI 子命令：`codex exec`、`codex review`、`codex mcp`、`codex plugin`
- 交互内 slash commands：`/model`、`/status`、`/review`、`/compact`

### 5.2 常见启动方式

- 普通进入交互界面：

```bash
codex
```

- 进入指定目录：

```bash
codex -C /path/to/project
```

- 启动时直接指定模型：

```bash
codex -m gpt-5.2-codex
```

- 启动时启用网页搜索：

```bash
codex --search
```

### 5.3 常用顶层 CLI 子命令

根据 `codex --help`，高频顶层命令包括：

- `codex exec`
  非交互方式运行任务
- `codex review`
  非交互方式做代码审查
- `codex login`
  登录管理
- `codex mcp`
  管理 MCP 服务器
- `codex plugin`
  管理插件
- `codex doctor`
  诊断安装、配置、认证和运行时健康
- `codex sandbox`
  在 Codex 提供的 sandbox 中运行命令
- `codex resume`
  恢复保存的交互会话
- `codex update`
  更新 Codex

### 5.4 一些关键全局参数

`codex --help` 里最重要的全局参数有：

- `-m, --model <MODEL>`
  指定模型
- `-s, --sandbox <SANDBOX_MODE>`
  指定沙箱模式
- `-a, --ask-for-approval <APPROVAL_POLICY>`
  指定 approval 策略
- `-C, --cd <DIR>`
  指定工作目录
- `--add-dir <DIR>`
  追加可写目录
- `-c, --config <key=value>`
  覆盖配置项
- `--enable <FEATURE>` / `--disable <FEATURE>`
  临时开关 feature
- `--search`
  为模型启用 live web search

## 6. Slash Commands：输入 `/` 弹出的命令菜单

以下内容基于本机 `codex-cli 0.144.1`，并结合 OpenAI 开源仓库中的 TUI 源码整理。不同版本可能略有差异。

### 6.1 说明

- 这是交互界面里输入 `/` 后出现的命令列表
- 不是 `codex --help` 里的顶层 CLI 子命令
- 一部分命令有别名
- 一部分命令支持参数

### 6.2 命令总表

| 命令 | 选项/参数 | 作用 | 备注 |
|---|---|---|---|
| `/model` | 无 | 选择模型和推理强度 | 还会带出动态 service-tier 命令 |
| `/ide` | `[on\|off\|status]` | 开/关/查看 IDE 上下文注入 | 包括当前选中内容、打开文件等 |
| `/permissions` | 无 | 配置 Codex 能做什么 | 常见是只读、workspace、full access |
| `/keymap` | `[debug]` | 配置快捷键 | `debug` 用于看按键识别 |
| `/vim` | 无 | 切换输入框 Vim 模式 | 作用于 composer |
| `/setup-default-sandbox` | 无 | 设置提升后的 agent sandbox | 主要是 Windows 特殊场景 |
| `/sandbox-add-read-dir` | `<absolute-directory-path>` | 给 sandbox 追加可读目录 | 平台可见性有差异 |
| `/experimental` | 无 | 开关实验特性 | 会写入配置 |
| `/approve` | 无 | 批准最近一次 auto-review 拒绝后的重试 | 内部名是 `AutoReview` |
| `/memories` | 无 | 配置 memory 的使用和生成 | 需要相应 feature |
| `/skills` | 无 | 查看/使用 skills | |
| `/import` | 无 | 从 Claude Code 导入设置/项目/最近会话 | |
| `/hooks` | 无 | 查看和管理 lifecycle hooks | |
| `/review` | `[自定义说明]` | 审查当前改动 | 带参数时是自定义 review 指令 |
| `/rename` | `[线程名]` | 重命名当前会话 | 不带参数会弹输入框 |
| `/new` | 无 | 新开一个聊天 | |
| `/archive` | 无 | 归档当前会话并退出 | |
| `/delete` | 无 | 永久删除当前会话并退出 | 不可恢复 |
| `/resume` | `[session-id 或 session-name]` | 恢复旧会话 | 不带参数会打开选择器 |
| `/fork` | 无 | fork 当前会话 | |
| `/app` | 无 | 在 Codex Desktop 继续当前会话 | 仅桌面端平台可见 |
| `/init` | 无 | 生成 `AGENTS.md` 指南文件 | |
| `/compact` | 无 | 压缩上下文，防止吃满 context | |
| `/plan` | `[消息]` | 切到 Plan mode | 带参数时直接把消息发过去 |
| `/goal` | `[objective\|clear\|edit\|pause\|resume]` | 设置或修改长任务目标 | 参数最完整的命令之一 |
| `/agent` | 无 | 切换当前 agent thread | |
| `/subagents` | 无 | 切换当前 agent thread | 与 `/agent` 同类 |
| `/side` | `[消息]` | 开 side conversation | |
| `/btw` | `[消息]` | 开 side conversation | 与 `/side` 等价 |
| `/copy` | 无 | 把上一条 agent 回复复制为 Markdown | |
| `/raw` | `[on\|off]` | 切换 raw scrollback 模式 | 方便复制纯文本 |
| `/diff` | 无 | 显示 git diff | 包含 untracked files |
| `/mention` | 无 | 插入文件 mention | 实际效果类似输入 `@` |
| `/status` | 无 | 查看当前会话配置和 token 用量 | 高频命令 |
| `/usage` | `[daily\|weekly\|cumulative]` | 查看账户 usage / reset | 需要 ChatGPT 登录 |
| `/debug-config` | 无 | 查看配置层与 requirement 来源 | 排障用 |
| `/title` | 无 | 配置 terminal title 显示项 | |
| `/statusline` | 无 | 配置状态栏显示项 | |
| `/theme` | 无 | 选择语法高亮主题 | |
| `/pets` | `[pet-id\|off\|none\|hide\|disable]` | 选择或关闭终端宠物 | `/pet` 也可识别 |
| `/mcp` | `[verbose]` | 列出 MCP 工具 | `verbose` 看详细信息 |
| `/apps` | 无 | 管理 apps/connectors | 需启用相关 feature |
| `/plugins` | 无 | 浏览 plugins | 需启用相关 feature |
| `/logout` | 无 | 登出 Codex | |
| `/quit` | 无 | 退出 Codex | |
| `/exit` | 无 | 退出 Codex | 与 `/quit` 同义 |
| `/feedback` | 无 | 上传日志/反馈给维护方 | |
| `/rollout` | 无 | 打印 rollout 文件路径 | 调试用途 |
| `/ps` | 无 | 列出后台 terminal | |
| `/stop` | 无 | 停掉所有后台 terminal | `/clean` 会映射到它 |
| `/clear` | 无 | 清屏并开启新 chat | |
| `/personality` | 无 | 选择 Codex 沟通风格 | 需相应 feature |
| `/test-approval` | 无 | 测试 approval request | 调试用途 |
| `/debug-m-drop` | 无 | 内部 memory 调试命令 | 不建议使用 |
| `/debug-m-update` | 无 | 内部 memory 调试命令 | 不建议使用 |

### 6.3 常见别名与等价关系

- `/side` 和 `/btw` 等价
- `/quit` 和 `/exit` 等价
- `/pets` 的别名是 `/pet`
- `/stop` 的别名是 `/clean`
- `/approve` 对应的是 auto-review retry，不是通用审批入口

### 6.4 使用时的注意事项

- `/usage` 依赖 ChatGPT 登录
- `/apps`、`/plugins`、`/personality` 等命令受 feature flag 影响
- `/app` 仅在部分平台可见
- side conversation 中只允许部分命令继续使用
- 任务执行过程中，一部分命令会被临时禁用

## 7. 常用工作流

这一章按“第一次上手”、“日常开发”、“长对话维护”和“代码审查”四类整理。

### 7.1 第一次上手

1. 安装并启动 `codex`
2. 完成 ChatGPT 登录或 API Key 登录
3. 进入项目目录后运行：

```bash
codex
```

4. 先执行几条基础命令确认状态：

- `/status`
- `/permissions`
- `/model`

5. 如果仓库还没有给 Codex 的项目说明，可以运行：

- `/init`

### 7.2 日常开发

典型路径：

1. 用自然语言描述任务
2. 需要切换模型时用 `/model`
3. 需要看上下文和用量时用 `/status`
4. 需要看当前改动时用 `/diff`
5. 需要指向具体文件时用 `/mention`

适合高频记住的命令：

- `/model`
- `/status`
- `/diff`
- `/review`
- `/compact`

### 7.3 长对话维护

会话变长后，优先使用：

- `/compact`
  总结历史，释放上下文
- `/goal`
  为长任务记录目标状态
- `/plan`
  切换到计划模式
- `/side`
  针对旁支问题开临时分支会话

### 7.4 代码审查工作流

交互式：

1. 完成修改后运行 `/review`
2. 如果有额外约束，可直接输入：

```text
/review 重点看回归风险、边界条件和测试覆盖
```

非交互式：

```bash
codex review
```

### 7.5 会话管理工作流

- `/new`
  新开聊天
- `/resume`
  恢复历史会话
- `/fork`
  从当前会话派生分支
- `/archive`
  归档退出
- `/delete`
  永久删除退出

## 8. 配置、权限与沙箱

这一部分直接影响 Codex 在本地能做什么，是使用时最需要有边界感的一章。

### 8.1 配置入口

CLI 帮助里明确提到默认配置文件位置：

- `~/.codex/config.toml`

也支持通过命令行临时覆盖：

```bash
codex -c model="o3"
```

```bash
codex -c 'sandbox_permissions=["disk-full-read-access"]'
```

还可以通过 profile 叠加配置：

```bash
codex -p myprofile
```

### 8.2 沙箱模式

`codex --help` 里可以确认 3 个沙箱模式：

- `read-only`
  只能读，修改和更高权限操作会受限
- `workspace-write`
  可以修改工作区文件
- `danger-full-access`
  可不受限访问更广泛文件与网络，风险最高

示例：

```bash
codex -s workspace-write
```

### 8.3 Approval 策略

CLI 当前支持 3 种 approval policy：

- `untrusted`
  只有“受信任命令”可直接执行，其他命令要审批
- `on-request`
  模型自行决定何时发起审批
- `never`
  不向用户请求审批，失败直接返回给模型

示例：

```bash
codex -a untrusted
```

### 8.4 搜索与网络

- `--search`
  启用 live web search

这和本地 shell 网络权限不是一回事。一个是模型可调用网页搜索能力，一个是本机命令/沙箱本身的网络边界。

### 8.5 高风险选项

CLI 里明确标注了两个高风险入口：

- `--dangerously-bypass-approvals-and-sandbox`
- `--dangerously-bypass-hook-trust`

含义：

- 跳过审批与沙箱，会显著增加误改文件、越权访问、泄漏数据的风险
- 只适合已经被外部环境严格隔离的自动化场景

### 8.6 使用建议

- 个人日常开发默认优先 `workspace-write + on-request`
- 不熟悉仓库时优先用 `/status` 和 `/permissions` 看当前状态
- 只有在明确知道风险和边界时，才使用 full access

## 9. MCP / Apps / Plugins

这三类能力都属于“让 Codex 接外部能力”，但定位不完全一样。

### 9.1 MCP 是什么

MCP 可以理解为给模型接入外部工具、资源、模板的一层协议。对 `Codex CLI` 来说，MCP 主要用于：

- 列出工具
- 调用工具
- 读取资源
- 接第三方服务

### 9.2 `/mcp` 和 `codex mcp` 的区别

- `/mcp`
  交互会话内查看当前已配置 MCP 工具
- `codex mcp`
  顶层管理命令，用于真正增删改查 MCP server 配置

`codex mcp --help` 里确认支持：

- `list`
- `get`
- `add`
- `remove`
- `login`
- `logout`

### 9.3 Apps

`/apps` 是交互界面里的入口，偏向“在当前会话中管理可用的 app / connector”。

它的特点：

- 依赖 feature 开启
- 更偏 TUI 内选择与调用
- 和 MCP 有交叉，但不等价

### 9.4 Plugins

`/plugins` 是交互界面的插件浏览入口。

顶层命令 `codex plugin` 当前支持：

- `add`
  从 marketplace snapshot 安装插件
- `list`
  列出插件
- `marketplace`
  管理 marketplace
- `remove`
  移除已安装插件

适合理解成：

- MCP 更像工具协议接入层
- Apps 更像面向会话的连接器入口
- Plugins 更像可安装扩展能力包

### 9.5 什么时候用哪一个

- 想管理 MCP server 配置：用 `codex mcp ...`
- 想在当前会话里查看 MCP 工具：用 `/mcp`
- 想浏览可接入应用：用 `/apps`
- 想安装或管理扩展包：用 `codex plugin ...` 或 `/plugins`

## 10. 排障与注意事项

这一部分记录实际使用里最常见的问题类型。

### 10.1 本地数据库只读或损坏

如果启动交互界面时报类似错误：

- `attempt to write a readonly database`
- `failed to initialize sqlite local db`

通常说明：

- `CODEX_HOME` 目录不可写
- 本地状态数据库权限异常
- 当前运行环境把 `~/.codex` 挂成了只读

排查建议：

1. 运行：

```bash
codex doctor
```

2. 检查 `~/.codex` 是否可写
3. 如在受限环境里运行，改用一个可写的 `CODEX_HOME`

### 10.2 命令没显示

如果 `/` 菜单里看不到某个命令，常见原因有：

- 平台不支持
- 未登录 ChatGPT
- 相关 feature 未启用
- 当前在 side conversation 中
- 当前有任务正在运行

例如：

- `/usage` 依赖 ChatGPT 登录
- `/app` 只在部分桌面平台可见
- `/plugins`、`/apps` 依赖 feature

### 10.3 登录相关问题

如果登录后行为异常，先检查：

- `codex login status`
- 当前是否混用了 ChatGPT 登录和 API Key 登录
- 环境变量是否污染了当前会话

### 10.4 配置不生效

常见原因：

- `config.toml` 字段名版本不兼容
- profile 叠加覆盖了基础配置
- 启动参数 `-c` 临时覆盖了文件配置

建议：

- 用 `/debug-config` 看实际生效配置层
- 必要时加 `--strict-config` 让未知字段直接报错

### 10.5 版本不一致

如果文档、机器行为、开源源码对不上，优先确认：

```bash
codex -V
which codex
```

尤其是下面几种情况容易混：

- `npm` 安装版
- `brew` 安装版
- 手工下载二进制版

### 10.6 权限过大带来的风险

需要特别警惕：

- `danger-full-access`
- `--dangerously-bypass-approvals-and-sandbox`
- 信任来源不明的 hooks / plugins / MCP 配置

这类配置在自动化里很方便，但在日常开发机上要慎用。

## 11. 参考链接

- Codex 仓库：https://github.com/openai/codex
- Codex CLI 文档：https://developers.openai.com/codex/cli
- Codex 鉴权文档：https://developers.openai.com/codex/auth
- Codex MCP 文档：https://developers.openai.com/codex/mcp
- Slash command 定义：
  https://raw.githubusercontent.com/openai/codex/main/codex-rs/tui/src/slash_command.rs
- Slash command 过滤逻辑：
  https://raw.githubusercontent.com/openai/codex/main/codex-rs/tui/src/bottom_pane/slash_commands.rs
- Slash command 分发逻辑：
  https://raw.githubusercontent.com/openai/codex/main/codex-rs/tui/src/chatwidget/slash_dispatch.rs
