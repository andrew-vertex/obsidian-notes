---
tags:
  - ai-skills
  - coding-agent
  - developer-tools
  - index
---
# AI Skills 笔记索引与整理规范

统一收纳各类 AI Coding Agent 的 `skills`、`plugins`、`rules`、`AGENTS.md` 指令模板，以及跨 Agent 的安装和使用说明。

## 目录定位

| 项目 | 说明 |
| --- | --- |
| 目录 | `Tools/AI/Skills/` |
| 收纳范围 | `Claude Code`、`Codex`、`OpenCode`、`Cursor`、`Windsurf`、`Cline` 等 agent 的 skills / 插件 / rules |
| 文档目标 | 说明“这是什么、解决什么问题、如何安装、如何使用、不同 Agent 有何差异、最佳实践是什么” |
| 优先信息源 | 官方文档、官方 GitHub README、官方安装脚本、官方 marketplace / skills 说明 |

## 命名规范

| 类型 | 命名建议 | 示例 |
| --- | --- | --- |
| 单个 skill / 插件文档 | `{name}-installation-usage-best-practices.md` | `caveman-installation-usage-best-practices.md` |
| 索引文档 | `README.md` | 当前文件 |
| 对比文档 | `{topic}-comparison.md` | `token-compression-skills-comparison.md` |

## 单篇文档建议结构

| 顺序 | 内容 |
| --- | --- |
| 1 | 项目定位与一句话总结 |
| 2 | `Key Takeaways` |
| 3 | 核心作用与适用场景 |
| 4 | macOS 安装前准备 |
| 5 | 安装方式总览 |
| 6 | `Claude Code / Codex / OpenCode` 差异 |
| 7 | 常用命令与触发方式 |
| 8 | always-on 配置方法 |
| 9 | 卸载 / 停用 |
| 10 | 参考来源 |

## 当前收录

| 名称 | 主题 | 状态 | 文档 |
| --- | --- | --- | --- |
| caveman | 输出压缩 / token 节省 / terse 模式 | 已整理 | [caveman-installation-usage-best-practices.md](./caveman-installation-usage-best-practices.md) |
| graphify | 知识图谱 / 代码库理解 / 多模态语料导航 | 已整理 | [graphify-installation-usage-best-practices.md](./graphify-installation-usage-best-practices.md) |

## 后续整理建议

| 建议 | 说明 |
| --- | --- |
| 先整理高频 skill | 先做你会反复装和反复切换的 skill |
| 每篇都写 Agent 差异表 | 避免只记安装命令，不记行为差异 |
| 统一标注核对日期 | skills 更新快，必须写“截至哪天” |
| 区分“官方支持”和“社区实践” | 避免把 repo 里的技巧误记成 agent 官方能力 |
