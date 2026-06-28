---
title: "Wiki Scaffold 优化设计文档"
created: 2026-06-28
description: "基于四合一原则的个人第二大脑 scaffold 重设计"
tags: [meta, design, scaffold]
status: finalized
---

# Wiki Scaffold 优化设计文档

## 背景

基于 claude-obsidian 插件搭建个人第二大脑（学习笔记、工作项目、日常思考）。初次 scaffold 使用了默认模板，但其结构与实际笔记风格差距大，需要优化。

## 设计原则（四合一）

1. **渐进式摘要** — Layer 0-4 渐次加深，大多数笔记停留 Layer 1
2. **模板即结构契约** — section 序列固定，元素按需裁减
3. **MOC 驱动结构** — 🎯📚🔮 三段式，主动策划而非被动列表
4. **可扫读性** — 执行摘要在前，表格/emoji 做视觉锚点

## 实施清单

- [ ] 7 个 `_templates/` 文件重写
- [ ] `wiki/overview.md` → Home MOC
- [ ] `wiki/hot.md` → emoji 锚点结构
- [ ] `wiki/log.md` → 表格 + 详情双层
- [ ] 4 个领域 MOC 重写
- [ ] 保留 `wiki/index.md` 作 AI 检索用
- [ ] 3-4 个典范页面
- [ ] 更新 AGENTS.md
