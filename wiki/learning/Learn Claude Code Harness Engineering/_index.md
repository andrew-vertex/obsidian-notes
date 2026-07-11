---
title: "Learn Claude Code Harness Engineering"
created: 2026-07-05
description: "系统学习 learn-claude-code 项目的课程型笔记入口"
tags: [learning, course, ai-agent, harness-engineering]
course: "Learn Claude Code"
platform: "GitHub repository"
progress: "0/20"
layer: 1
status: developing
---

# Learn Claude Code Harness Engineering

> 一句话概述：通过 `learn-claude-code` 项目系统理解 Claude Code 风格 agent harness 的组成、演进路径和工程取舍。

## 学习入口

- [[00-学习总览]] — 学习目标、节奏、输出物
- [[01-章节笔记目录]] — 20 章逐章笔记入口
- [[02-源码阅读路线]] — 从 README、`code.py` 到 `agents/` 的阅读顺序
- [[03-概念地图]] — harness engineering 核心概念关系
- [[04-问题与实验]] — 学习过程中的问题、实验和验证记录

## 学习进度

| 章节 | 状态 | 主题 | 关键收获 |
|------|------|------|---------|
| [[chapters/s01-agent-loop|s01 Agent Loop]] | ⬜ 未开始 | 一个工具调用循环如何构成最小 agent | |
| [[chapters/s02-tool-use|s02 Tool Use]] | ⬜ 未开始 | 工具注册、dispatch map 与 tool_result 回填 | |
| [[chapters/s03-permission|s03 Permission]] | ⬜ 未开始 | 权限边界、风险判断与审批点 | |
| [[chapters/s04-hooks|s04 Hooks]] | ⬜ 未开始 | 在循环外扩展生命周期行为 | |
| [[chapters/s05-todowrite|s05 TodoWrite]] | ⬜ 未开始 | 用任务清单稳定多步骤执行 | |
| [[chapters/s06-subagent|s06 Subagent]] | ⬜ 未开始 | 用隔离上下文拆分大任务 | |
| [[chapters/s07-skill-loading|s07 Skill Loading]] | ⬜ 未开始 | 按需加载知识和能力 | |
| [[chapters/s08-context-compact|s08 Context Compact]] | ⬜ 未开始 | 上下文压缩与信息保真 | |
| [[chapters/s09-memory|s09 Memory]] | ⬜ 未开始 | 跨会话记忆与可检索知识 | |
| [[chapters/s10-system-prompt|s10 System Prompt]] | ⬜ 未开始 | 系统提示词作为 harness 契约 | |
| [[chapters/s11-error-recovery|s11 Error Recovery]] | ⬜ 未开始 | 错误识别、恢复策略与重试边界 | |
| [[chapters/s12-task-system|s12 Task System]] | ⬜ 未开始 | 持久化任务、依赖与状态流转 | |
| [[chapters/s13-background-tasks|s13 Background Tasks]] | ⬜ 未开始 | 异步任务与前后台协作 | |
| [[chapters/s14-cron-scheduler|s14 Cron Scheduler]] | ⬜ 未开始 | 定时触发与周期性 agent 工作 | |
| [[chapters/s15-agent-teams|s15 Agent Teams]] | ⬜ 未开始 | 多 agent 分工与结果汇总 | |
| [[chapters/s16-team-protocols|s16 Team Protocols]] | ⬜ 未开始 | 团队协议、消息格式与协作约束 | |
| [[chapters/s17-autonomous-agents|s17 Autonomous Agents]] | ⬜ 未开始 | 自主循环、停止条件与监督 | |
| [[chapters/s18-worktree-isolation|s18 Worktree Isolation]] | ⬜ 未开始 | 用 worktree 隔离并行任务 | |
| [[chapters/s19-mcp-plugin|s19 MCP Plugin]] | ⬜ 未开始 | 外部能力接入与插件边界 | |
| [[chapters/s20-comprehensive-agent|s20 Comprehensive Agent]] | ⬜ 未开始 | 整合完整 harness 的端到端实现 | |

## 关联

- [[learning/_index|学习 MOC]] — 本课程所在的上层导航
- [[AI Agent 的 Harness Engineering]] — 相关概念背景
- [[learning/Git 实用技巧：worktree、rebase 与日常分支管理|Git worktree]] — s18 的前置技能

## 参考来源

- `/Users/yuanjianwei/workspace/learning/learn-claude-code`
