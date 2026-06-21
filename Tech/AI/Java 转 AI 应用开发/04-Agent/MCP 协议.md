---
title: MCP 协议
created: 2026-06-21
description: MCP 的 Client、Server、Host、Transport 以及 Tools、Resources、Prompts 原语。
tags: [ai, agent, mcp, protocol, planned]
status: planned
---
# MCP 协议

> 阅读状态：待精读。协议与 SDK 演进较快，内容需记录规范版本和验证日期。

[TOC]

## Key Takeaways

- 待读完来源并核验当前规范后填写。

## 阅读问题

- Host、Client、Server 和 Transport 的职责是什么？
- Tools、Resources、Prompts 如何区分？
- MCP 标准化了什么，没有替代哪些鉴权、授权和业务控制？

## 待整理大纲

1. 协议目标与架构角色
2. 生命周期和能力协商
3. 三类原语
4. Transport、JSON-RPC 与错误模型
5. Java 生态、安全和生产治理

## 实践与验证

- [ ] 实现或接入一个只读 MCP Server。
- [ ] 验证能力发现、参数错误、超时和权限拒绝。

## 相关页面

- [[../01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地]]
- [[Agent Skills]]
- [[AI Agent 基础]]

## 参考来源

- <https://javaguide.cn/ai/agent/mcp.html>
