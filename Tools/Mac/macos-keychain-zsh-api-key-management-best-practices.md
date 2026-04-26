---
tags:
  - mac
  - macos
  - keychain
  - zsh
  - api-key
  - security
---
# macOS Keychain + zsh 管理 API Key 最佳实践

面向 macOS 本地开发环境，整理 API Key 从明文环境变量迁移到 `Keychain/钥匙串` 的推荐做法，适用于 `DeepSeek`、`DashScope`、`OpenAI`、`Gemini` 等常见 provider。

截至 `2026-04-26`，本文基于 macOS 自带 `security` 命令和 zsh 常见启动流程整理。

## 目录

- [Key Takeaways](#key-takeaways)
- [为什么不要把 API Key 直接写在 shell 配置里](#为什么不要把-api-key-直接写在-shell-配置里)
- [推荐方案](#推荐方案)
- [推荐命名规范](#推荐命名规范)
- [落地步骤](#落地步骤)
- [推荐的 zsh 配置](#推荐的-zsh-配置)
- [日常操作](#日常操作)
- [验证方式](#验证方式)
- [常见坑](#常见坑)
- [最佳实践清单](#最佳实践清单)

## Key Takeaways

- 不要把 `sk-...` 这类 API Key 明文写进 `~/.zshrc`、`~/.bashrc`、项目 `.env` 或脚本里。
- macOS 本地环境优先用 `Keychain` 保存密钥，再由 shell 在启动时按需读取。
- 环境变量名保持标准化，例如 `DEEPSEEK_API_KEY`、`DASHSCOPE_API_KEY`，不要用 `deepseek=...` 这种随意命名。
- `~/.zshrc` 只负责“读取”，不负责“存储”。真实密钥应只出现在 Keychain 中。
- 明文的 `~/.zshrc.secrets` 可以保留，但应只作为临时覆盖入口，不应长期保存真实密钥。

## 为什么不要把 API Key 直接写在 shell 配置里

把密钥直接写进 shell 配置有几个典型问题：

- 终端配置文件经常被同步、备份、截图、复制，泄漏概率很高。
- 新旧机器迁移时，明文密钥很容易混进 dotfiles 仓库。
- 后续排查问题时，`cat ~/.zshrc`、分享配置片段、录屏演示都可能暴露真实 key。
- 一旦多人共用模版文件，复制粘贴会把本机私钥带到别处。

一句话理解：

- shell 配置应该保存“如何读取密钥”，而不是“密钥本身”。

## 推荐方案

推荐结构如下：

1. 用 macOS Keychain 保存每个 provider 的 API Key。
2. 在 `~/.zshrc` 中封装一个 `keychain_get()` 函数。
3. 用标准环境变量名从 Keychain 读取值。
4. 保留 `~/.zshrc.secrets`，但只用于临时覆盖或非长期敏感项。

推荐的数据流：

`Keychain -> ~/.zshrc -> export PROVIDER_API_KEY -> CLI / SDK / scripts`

## 推荐命名规范

建议统一两套命名：

| 用途 | 命名规则 | 示例 |
| --- | --- | --- |
| 环境变量名 | 全大写 + 下划线 | `DEEPSEEK_API_KEY` |
| Keychain service 名 | 小写 + 下划线 | `deepseek_api_key` |

推荐映射：

| Provider | Keychain service | 环境变量 |
| --- | --- | --- |
| DeepSeek | `deepseek_api_key` | `DEEPSEEK_API_KEY` |
| DashScope | `dashscope_api_key` | `DASHSCOPE_API_KEY` |
| OpenAI | `openai_api_key` | `OPENAI_API_KEY` |
| Gemini | `gemini_api_key` | `GEMINI_API_KEY` |

这样做的好处是：

- 一眼能看出变量来源
- 后续新增 provider 时不需要重新设计命名
- shell、脚本、SDK 配置保持一致

## 落地步骤

### 1. 写入或更新 Keychain

写入新 key：

```bash
security add-generic-password -a "$(id -un)" -s "deepseek_api_key" -w 'sk-xxxxx' -U
security add-generic-password -a "$(id -un)" -s "dashscope_api_key" -w 'sk-xxxxx' -U
```

说明：

- `-a "$(id -un)"` 使用当前 macOS 用户名作为账号标识
- `-s` 是这条密码在 Keychain 中的 service 名
- `-w` 是密码内容
- `-U` 表示存在则更新，不存在则创建

### 2. 在 `~/.zshrc` 中按需读取

推荐实现：

```bash
keychain_get() {
  local service="$1"
  local account="${KEYCHAIN_ACCOUNT:-${USER:-${LOGNAME:-$(id -un)}}}"

  security find-generic-password -a "$account" -s "$service" -w 2>/dev/null \
    || security find-generic-password -s "$service" -w 2>/dev/null
}

export DEEPSEEK_API_KEY="${DEEPSEEK_API_KEY:-$(keychain_get deepseek_api_key)}"
export DASHSCOPE_API_KEY="${DASHSCOPE_API_KEY:-$(keychain_get dashscope_api_key)}"
```

这个写法有几个优点：

- 优先保留外部显式传入的环境变量，不会强行覆盖
- `USER`、`LOGNAME` 缺失时仍能回退到 `id -un`
- 先按 `account + service` 查找，再退回按 `service` 查找，兼容性更好

### 3. 处理 `~/.zshrc.secrets`

推荐把它降级成“临时覆盖文件”，例如：

```bash
# Sensitive values should live in macOS Keychain and be loaded by ~/.zshrc.
# Keep this file only for temporary local overrides that should not be committed.
```

不要再保留：

```bash
export DEEPSEEK_API_KEY="sk-xxxxx"
export DASHSCOPE_API_KEY="sk-xxxxx"
```

## 推荐的 zsh 配置

如果你已经有如下结构：

- `~/.zshrc`
- `~/.zshrc.secrets`

那么推荐职责划分是：

- `~/.zshrc`：通用启动逻辑、PATH、工具初始化、从 Keychain 读取密钥
- `~/.zshrc.secrets`：极少数临时覆盖项，不保存长期 API Key

这比把所有东西都堆到 `~/.zshrc` 更可维护，也更容易审计。

## 日常操作

### 新增一个 provider

```bash
security add-generic-password -a "$(id -un)" -s "openai_api_key" -w 'sk-xxxxx' -U
```

然后在 `~/.zshrc` 加一行：

```bash
export OPENAI_API_KEY="${OPENAI_API_KEY:-$(keychain_get openai_api_key)}"
```

### 轮换 key

同样使用 `-U` 直接覆盖：

```bash
security add-generic-password -a "$(id -un)" -s "deepseek_api_key" -w 'sk-new-xxxxx' -U
```

### 删除旧 key

```bash
security delete-generic-password -s "deepseek_api_key"
```

如果你使用了 `account` 维度，也可以加上 `-a "$(id -un)"`。

## 验证方式

### 1. 验证 Keychain 中是否存在

```bash
security find-generic-password -s "deepseek_api_key" -w >/dev/null && echo ok
```

### 2. 验证新 shell 是否能自动加载

```bash
zsh -ic 'printf "deepseek_len=%s\n" "${#DEEPSEEK_API_KEY}"'
```

建议验证“长度”或“是否为空”，不要直接 `echo "$DEEPSEEK_API_KEY"`。

### 3. 在脚本中使用

```bash
curl -H "Authorization: Bearer $DEEPSEEK_API_KEY" ...
```

## 常见坑

### 1. `~/.zshrc` 不是所有 shell 都会读取

`~/.zshrc` 主要面向交互式 shell。

如果某些自动化任务、GUI 应用、定时任务或服务进程拿不到变量，不一定是 Keychain 有问题，可能只是它根本没有加载 `~/.zshrc`。

应对方式：

- 对交互式终端场景，继续用 `~/.zshrc`
- 对脚本场景，显式 `source ~/.zshrc`
- 对独立服务进程，优先用服务自身的 secret 配置能力，不要盲目依赖 shell 环境变量

### 2. 直接把 key 打印到终端

以下做法风险很高：

```bash
echo "$DEEPSEEK_API_KEY"
printenv DEEPSEEK_API_KEY
```

更推荐：

```bash
[[ -n "$DEEPSEEK_API_KEY" ]] && echo loaded
printf "%s\n" "${#DEEPSEEK_API_KEY}"
```

### 3. 命名不统一

例如：

- `deepseek=...`
- `deepseek_key=...`
- `DS_API=...`

这类命名在脚本迁移、团队协作、CLI 接入时都会增加摩擦。统一成 `PROVIDER_API_KEY` 最省事。

### 4. 把 `.zshrc.secrets` 当长期密码仓库

它只是比 `~/.zshrc` 更不显眼，不代表安全。

本质上它依然是本地明文文件，不应作为长期 API Key 存储位置。

## 最佳实践清单

- 所有长期 API Key 存 Keychain，不存 dotfiles 明文
- 环境变量统一用 `PROVIDER_API_KEY`
- Keychain service 统一用 `provider_api_key`
- `~/.zshrc` 只负责读取，不保存真实密钥
- `~/.zshrc.secrets` 只做临时覆盖，不做长期存储
- 使用 `-U` 更新已有 key，避免重复记录
- 验证时只检查长度或存在性，不直接打印值
- 对非交互式脚本显式加载环境，别假设它会自动读 `~/.zshrc`
- key 泄漏后先在 provider 后台吊销，再更新本地 Keychain

