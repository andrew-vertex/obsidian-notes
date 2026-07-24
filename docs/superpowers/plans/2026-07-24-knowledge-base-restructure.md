# 知识库结构重构与中文化 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将个人 Obsidian 知识库重构为“极简 5 文件 wiki 控制层 + 5 大中文主题目录”的清晰架构，并更新 AGENTS.md 规范，全面从“复杂 AI 实体抽卡”转向“高效笔记整理”。

**Architecture:** 
1. 建立 5 大中文主题目录与 1 个收集箱。
2. 逐一平滑迁移存量 Markdown 笔记到对应中文子目录中，确保零文件丢失。
3. 使用 macOS `trash` 命令安全清除旧 `_templates/` 和旧 `wiki/` 及其无用碎片。
4. 全新重建极简 `wiki/` 控制层（仅包含 `overview.md`, `hot.md`, `goals.md`, `index.md`, `log.md` 5 个文件）。
5. 重构 `AGENTS.md` 确定新的 AI 规范与交互契约。

**Tech Stack:** macOS CLI (`mkdir`, `mv`, `trash`, `git`), Markdown, Obsidian Standard.

## Global Constraints

- 所有删除/清理操作严禁使用 `rm`，必须使用 macOS 命令 `trash`。
- 所有原有的 Markdown 笔记在迁移中不得损坏或丢失。
- 最终根目录下除 5 大中文主题目录、`6. 收集箱`、`wiki/` 和 `AGENTS.md` / `skills-lock.json` 外，不留任何散乱英文分类文件夹。

---

### Task 1: 创建全新的中文主目录与子目录结构

**Files:**
- Create: `1. 技术研发/AI应用开发/`
- Create: `1. 技术研发/后端与架构/`
- Create: `1. 技术研发/DevOps与运维/`
- Create: `1. 技术研发/算法与编程/`
- Create: `2. 实用工具/AI工具手册/`
- Create: `2. 实用工具/Mac效率工具/`
- Create: `3. 工作与业务/保险核赔业务/`
- Create: `3. 工作与业务/工单与排障/`
- Create: `3. 工作与业务/履历与项目经历/`
- Create: `4. 思考与认知/思维模型/`
- Create: `4. 思考与认知/语言学习/`
- Create: `4. 思考与认知/读书与感悟/`
- Create: `5. 项目与作品/博客与文章/`
- Create: `6. 收集箱/`

**Interfaces:**
- Consumes: 无
- Produces: 完整的中文知识库目录基建

- [ ] **Step 1: 创建分类子目录**

Run:
```bash
mkdir -p "1. 技术研发/AI应用开发" "1. 技术研发/后端与架构" "1. 技术研发/DevOps与运维" "1. 技术研发/算法与编程"
mkdir -p "2. 实用工具/AI工具手册" "2. 实用工具/Mac效率工具"
mkdir -p "3. 工作与业务/保险核赔业务" "3. 工作与业务/工单与排障" "3. 工作与业务/履历与项目经历"
mkdir -p "4. 思考与认知/思维模型" "4. 思考与认知/语言学习" "4. 思考与认知/读书与感悟"
mkdir -p "5. 项目与作品/博客与文章"
mkdir -p "6. 收集箱"
```

- [ ] **Step 2: 验证目录创建成功**

Run: `ls -d [1-6].*`
Expected: 对应 6 个中文顶级目录列出。

---

### Task 2: 平滑迁移现有笔记文件至中文目录

**Files:**
- Move: `Tech/`, `AI/` $\rightarrow$ `1. 技术研发/`
- Move: `Tools/` $\rightarrow$ `2. 实用工具/`
- Move: `Work/`, `Resume/` $\rightarrow$ `3. 工作与业务/`
- Move: `Thinking/` $\rightarrow$ `4. 思考与认知/`
- Move: `Projects/` $\rightarrow$ `5. 项目与作品/`
- Move: `Clippings/`, `Inbox/` $\rightarrow$ `6. 收集箱/`

**Interfaces:**
- Consumes: Task 1 创建的中文目录
- Produces: 无损重排后的正式笔记库

- [ ] **Step 1: 迁移【技术研发】笔记**

Run:
```bash
mv AI/"AI Developer"/* "1. 技术研发/AI应用开发/" 2>/dev/null || true
mv Tech/AI/* "1. 技术研发/AI应用开发/" 2>/dev/null || true
mv Tech/Agent/* "1. 技术研发/AI应用开发/" 2>/dev/null || true
mv Tech/Architecture/* "1. 技术研发/后端与架构/" 2>/dev/null || true
mv Tech/Backend/* "1. 技术研发/后端与架构/" 2>/dev/null || true
mv Tech/DevOps/* "1. 技术研发/DevOps与运维/" 2>/dev/null || true
mv Tech/Programming/* "1. 技术研发/算法与编程/" 2>/dev/null || true
mv Tech/Algorithms/* "1. 技术研发/算法与编程/" 2>/dev/null || true
```

- [ ] **Step 2: 迁移【实用工具】笔记**

Run:
```bash
mv Tools/AI/* "2. 实用工具/AI工具手册/" 2>/dev/null || true
mv Tools/Mac/* "2. 实用工具/Mac效率工具/" 2>/dev/null || true
mv "Tools/claude-obsidian 插件使用指南.md" "2. 实用工具/Mac效率工具/" 2>/dev/null || true
```

- [ ] **Step 3: 迁移【工作与业务】笔记**

Run:
```bash
mv Work/design/* "3. 工作与业务/保险核赔业务/" 2>/dev/null || true
mv Work/development/* "3. 工作与业务/工单与排障/" 2>/dev/null || true
mv Work/skills/* "3. 工作与业务/工单与排障/" 2>/dev/null || true
mv Work/"Git 实用技巧：worktree、rebase 与日常分支管理.md" "2. 实用工具/Mac效率工具/" 2>/dev/null || true
mv Resume/project/* "3. 工作与业务/履历与项目经历/" 2>/dev/null || true
mv Resume/个人简历.md "3. 工作与业务/履历与项目经历/" 2>/dev/null || true
```

- [ ] **Step 4: 迁移【思考与认知】与【项目与作品】笔记**

Run:
```bash
mv "Thinking/Cognitive Upgrade/高杠杆思维模型实战手册.md" "4. 思考与认知/思维模型/" 2>/dev/null || true
mv "Thinking/Cognitive Upgrade/"*.md "4. 思考与认知/语言学习/" 2>/dev/null || true
mv Projects/Blog/* "5. 项目与作品/博客与文章/" 2>/dev/null || true
```

- [ ] **Step 5: 迁移【收集箱】草稿**

Run:
```bash
mv Clippings/* "6. 收集箱/" 2>/dev/null || true
mv Inbox/* "6. 收集箱/" 2>/dev/null || true
```

- [ ] **Step 6: 验证核心笔记完整存在于新中文路径中**

Run:
```bash
ls "1. 技术研发/AI应用开发/" "2. 实用工具/AI工具手册/" "3. 工作与业务/保险核赔业务/"
```

---

### Task 3: 使用 `trash` 安全清理废弃模板、旧 wiki 与废弃空目录

**Files:**
- Trash: `_templates/`
- Trash: `wiki/`
- Trash: 旧空目录 (`AI/`, `Archive/`, `Clippings/`, `Inbox/`, `Life/`, `Projects/`, `Resume/`, `Tech/`, `Thinking/`, `Tools/`, `Work/`)

**Interfaces:**
- Consumes: 完成迁移后的绝大多数空旧文件夹
- Produces: 干净清爽的根目录环境

- [ ] **Step 1: 使用 `trash` 清除旧 `_templates` 与旧 `wiki` 目录**

Run:
```bash
trash _templates wiki
```

- [ ] **Step 2: 使用 `trash` 清理已移动空旧目录**

Run:
```bash
trash AI Archive Clippings Inbox Life Projects Resume Tech Thinking Tools Work
```

- [ ] **Step 3: 检查根目录，确认只有全新的中文目录和必要的配置文件**

Run: `ls -la`
Expected: 只有 `1. 技术研发`, `2. 实用工具`, `3. 工作与业务`, `4. 思考与认知`, `5. 项目与作品`, `6. 收集箱`, `AGENTS.md`, `skills-lock.json`, `docs` 以及隐藏点目录。

---

### Task 4: 重新初始化极简 `wiki/` 5 文件控制层

**Files:**
- Create: `wiki/overview.md`
- Create: `wiki/hot.md`
- Create: `wiki/goals.md`
- Create: `wiki/index.md`
- Create: `wiki/log.md`

**Interfaces:**
- Consumes: 新的中文目录结构
- Produces: 极简 AI 上下文与索引控制层

- [ ] **Step 1: 创建 `wiki/` 目录**

Run: `mkdir -p wiki`

- [ ] **Step 2: 写入 `wiki/overview.md` (总览导航页)**

Create `wiki/overview.md`:
```markdown
---
title: "个人知识库总览"
created: 2026-07-24
updated: 2026-07-24
description: "个人第二大脑导航地图"
---

# 个人知识库总览

## 🗺️ 知识版图

| 领域 | 包含内容 | 目录入口 |
|------|---------|---------|
| 💻 技术研发 | AI应用开发、后端与架构、DevOps、算法与编程 | `1. 技术研发/` |
| 🔧 实用工具 | AI工具手册、Mac效率工具指南 | `2. 实用工具/` |
| 💼 工作与业务 | 保险理赔系统、工单与排障、个人履历与项目经历 | `3. 工作与业务/` |
| 💭 思考与认知 | 思维模型、英语语言学习、读书与感悟 | `4. 思考与认知/` |
| 🚀 项目与作品 | 个人作品、博客与文章 | `5. 项目与作品/` |
| 📥 收集箱 | 网页剪藏、草稿、临时资料收集 | `6. 收集箱/` |

## 🔗 控制工具
- [[hot|Hot Cache 动态上下文]]
- [[goals|长期目标与任务清单]]
- [[index|全库笔记索引与时间线]]
- [[log|知识变更日志]]
```

- [ ] **Step 3: 写入 `wiki/hot.md` (动态焦点)**

Create `wiki/hot.md`:
```markdown
---
title: "Hot Cache"
updated: 2026-07-24
---

# Hot Cache (动态上下文)

## 🎯 当前焦点
- 知识库完成结构重构：全面转为中文主题目录，专注于高效笔记整理与内容沉淀。
- 当前焦点：系统提升 AI 应用开发（Agent Harness / Spec Kit）与医保核赔业务排障效率。

## 📝 最近动态
- 2026-07-24: 重构目录结构为 5 大中文领域，清理废弃 `_templates` 与碎卡片，全新初始化极简 wiki 控制层。
```

- [ ] **Step 4: 写入 `wiki/goals.md` (目标管理)**

Create `wiki/goals.md`:
```markdown
---
title: "长期目标与任务管理"
created: 2026-07-24
---

# 长期目标与任务管理

## 🎯 核心目标
1. **AI 应用开发转型**：熟练掌握 Agentic 编程、Harness Engineering 与 SDD 规范驱动开发。
2. **工作业务精通**：完善医保核赔业务排障手册与系统设计知识沉淀。
3. **认知与语言提升**：持续积累高杠杆思维模型与英语能力。

## 📋 待整理/待补充清单
- [ ] 整理 `6. 收集箱/` 中的草稿笔记，归入对应的中文主题目录。
```

- [ ] **Step 5: 写入 `wiki/index.md` 与 `wiki/log.md`**

Create `wiki/log.md`:
```markdown
# 知识库变更日志

- 2026-07-24: 完成知识库结构重构，全面采用中文主题目录划分，建立极简 5 文件 wiki 控制层。
```

- [ ] **Step 6: 生成 `wiki/index.md` 扁平清单**

自动扫描当前中文目录生成 index 文件。

---

### Task 5: 重构 `AGENTS.md` 规范契约

**Files:**
- Overwrite: `AGENTS.md`

**Interfaces:**
- Consumes: 新结构规则
- Produces: 约束 Agent 行为的新协议

- [ ] **Step 1: 编写重构后的 `AGENTS.md`**

Overwrite `AGENTS.md`:
```markdown
# 知识库 Agent 规范

本仓库是个人 Obsidian 知识库。AI Agent 的核心任务是**协助用户进行笔记的精细整理、分类归档与索引同步**，不再进行多余的碎卡片提炼。

## 核心目录约法则

- `1. 技术研发/` — AI开发、后端架构、DevOps运维、算法编程。
- `2. 实用工具/` — AI工具手册、Mac效率指南。
- `3. 工作与业务/` — 保险核赔业务、工单排障、履历与项目经历。
- `4. 思考与认知/` — 思维模型、语言学习、读书与感悟。
- `5. 项目与作品/` — 个人项目、博客与文章。
- `6. 收集箱/` — 临时文件、草稿、剪藏入口。
- `wiki/` — 极简 AI 控制层（包含 `overview.md`, `hot.md`, `goals.md`, `index.md`, `log.md` 5 个控制文件）。

## 核心行为规范

1. **专注整理，禁止碎卡**：
   - 严禁自动生成 `concepts/` 或 `entities/` 碎片卡片。
   - 收到新资料或要求整理笔记时，必须将内容整理为**一整篇结构完整、带标题、表格与总结的 Markdown 笔记**，并直接放入对应的中文主题目录中。
2. **物理归位**：
   - 严禁将正式笔记乱丢到 `wiki/` 或 `6. 收集箱/`。
3. **控制层同步**：
   - 每次新增或更新笔记后，仅需更新 `wiki/index.md`（索引）与 `wiki/log.md`（日志），需要时更新 `wiki/hot.md`。
```

- [ ] **Step 2: Commit 最终修改**

Run:
```bash
git add .
git commit -m "refactor: complete knowledge base restructuring to Chinese directories and minimal wiki control layer"
```

---
