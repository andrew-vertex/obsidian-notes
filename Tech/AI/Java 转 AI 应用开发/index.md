---
title: Java 转 AI 应用开发索引
created: 2026-06-21
description: Java 后端开发者学习 LLM、RAG、Agent 与生产级 AI 应用工程的分阶段导航页。
tags:
  - index
  - ai
  - java
  - learning-roadmap
---
# Java 转 AI 应用开发索引

本目录将学习路线拆成可独立维护的专题笔记。先阅读 [[Java 后端转 AI 应用开发学习路线]]，再用 [[学习进度与实践清单]] 管理学习证据；专题页当前是待填充骨架，读完来源文章后逐步补全。

[TOC]

## 使用方式

1. 阶段 0～2 顺序推进，先建立模型边界和稳定调用能力。
2. 阶段 3 的 RAG 与阶段 4 的 Agent 分开练习，再在项目中融合。
3. 评测、安全、成本和可观测不是收尾项，应从第一个 Demo 开始记录。
4. 每完成一篇专题，填写 `Key Takeaways`、实验记录、失败案例和待确认问题。

## 路线入口

- [[Java 后端转 AI 应用开发学习路线]]：完整学习地图、阶段依赖和工程原则。
- [[学习进度与实践清单]]：以可运行代码、评测结果和架构决策作为完成标准。
- [[Tech/AI/AI 应用开发学习体系|AI 应用开发学习体系]]：旧版粗粒度主题总览。

## 阶段 0：转型定位

- [[00-转型定位/后端开发者转型 AI Agent]]：确认目标是 AI 应用工程，而非模型算法研发。

## 阶段 1：大模型基础与接入

- [[01-大模型基础/LLM 运行机制：Token、上下文窗口与采样参数]]
- [[01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地]]
- [[01-大模型基础/LLM API 调用工程]]
- [[01-大模型基础/Java AI 框架选型]]

## 阶段 2：Prompt 与上下文

- [[02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？]]
- [[02-Prompt 与上下文/上下文工程]]

## 阶段 3：RAG 与知识库

- [[03-RAG/RAG 基础]]
- [[03-RAG/RAG 文档处理与切分]]
- [[03-RAG/RAG 向量索引与向量数据库]]
- [[03-RAG/RAG 知识库更新]]
- [[03-RAG/RAG 检索优化]]
- [[03-RAG/GraphRAG]]

## 阶段 4：Agent 关键能力

- [[04-Agent/AI Agent 基础]]
- [[04-Agent/AI Agent 记忆系统]]
- [[04-Agent/Agent Skills]]
- [[04-Agent/MCP 协议]]
- [[04-Agent/Harness Engineering]]
- [[04-Agent/Workflow、Graph 与 Loop]]

## 阶段 5：生产工程化

- [[05-工程化/AI 应用系统设计]]
- [[05-工程化/大模型网关]]
- [[05-工程化/AI 应用评测体系]]
- [[05-工程化/AI 应用安全与合规]]

## 阶段 6：项目实战

- [[06-项目实战/企业知识库问答项目]]
- [[06-项目实战/多工具 Agent 项目]]
- [[06-项目实战/智能面试平台项目]]

## 阶段 7：按需进阶与复盘

- [[07-进阶与复盘/AI 语音技术]]
- [[07-进阶与复盘/AI 应用开发面试复盘]]

## 维护约定

| 状态 | 含义 | 完成条件 |
| --- | --- | --- |
| `planned` | 已建骨架，尚未精读 | 已记录来源与关键问题 |
| `reading` | 正在阅读和实验 | 有增量摘录、实验或疑问 |
| `draft` | 已形成初稿 | 主要章节完整且有来源 |
| `verified` | 已实践验证 | 有代码、评测、Trace 或项目证据 |

## 相关页面

- [[Tech/_kb/index|Tech 领域索引]]
- [[../../_kb/raw/JavaGuide Java Go 开发者 AI 应用开发路线来源记录|来源记录]]
- [[../../../Inbox/Java-to-AI-roadMap - AI 应用开发与 Agent 学习路线|原始阅读笔记]]

## 参考来源

- Java/Go 开发者 AI 应用开发与 Agent 学习路线：<https://javaguide.cn/roadmap/java-to-ai-roadmap.html>
- JavaGuide AI 应用开发知识体系：<https://javaguide.cn/ai/>
- 本地网页剪藏：[[../../../Clippings/JavaGo 开发者 AI 应用开发与 Agent 学习路线（2026 最新版）]]
