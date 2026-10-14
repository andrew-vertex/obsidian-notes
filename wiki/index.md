---
title: "全局笔记索引与清单"
description: "通过 Dataview 自动聚合的实时动态知识库索引"
updated: 2026-10-14
---

# 全局笔记索引与清单

> 💡 **自动索引说明**：本页面由 **Dataview** 插件动态聚合生成。任何新增或修改的笔记，只要归档在对应目录下，均会自动在此呈现，**无需手动或由 AI Agent 逐条维护链接表**。

## 🕒 最近更新 Top 10

```dataview
TABLE file.folder as "所在目录", default(updated, file.mday) as "更新时间", tags as "标签"
FROM "" AND !"wiki" AND !"6. 收集箱"
SORT default(updated, file.mday) desc
LIMIT 10
```

---

## 💻 1. 技术研发

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "1. 技术研发"
SORT default(updated, file.mday) desc
```

---

## 🔧 2. 实用工具

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "2. 实用工具"
SORT default(updated, file.mday) desc
```

---

## 💼 3. 工作与业务

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "3. 工作与业务"
SORT default(updated, file.mday) desc
```

---

## 💭 4. 思考与认知

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "4. 思考与认知"
SORT default(updated, file.mday) desc
```

---

## 🚀 5. 项目与作品

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "5. 项目与作品"
SORT default(updated, file.mday) desc
```

---

## 📥 6. 收集箱 (剪藏与临时输入)

```dataview
TABLE default(updated, file.mday) as "更新时间", tags as "标签"
FROM "6. 收集箱"
SORT file.mtime desc
```
