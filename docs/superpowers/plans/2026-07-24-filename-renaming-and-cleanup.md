# 知识库文件名规范化重命名 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 针对 1~6 中文目录下的所有 Markdown 笔记执行批量规范重命名（消除疑问句、去除口语化描述、规范大小写与中英文空格、简化罗列词汇），并自动全量重新生成 `wiki/index.md` 索引。

**Architecture:** 
1. 使用 `git mv` 安全批量重命名非规范笔记文件。
2. 运行 Python 自动化索引更新脚本，重新扫描更新 `wiki/index.md` 节点。
3. 提交 `git commit` 保存变更。

**Tech Stack:** Shell, Git (`git mv`), Python.

## Global Constraints

- 重命名操作优先使用 `git mv` 保留 Git 历史跟踪。
- 绝不改动正文核心知识，仅修正文件名。
- 重命名完成后立即重新生成 `wiki/index.md`，保证无死链。

---

### Task 1: 批量重命名非规范笔记文件

**Files:**
- Rename: 1. 技术研发/、2. 实用工具/、3. 工作与业务/、6. 收集箱/ 中的乱象文件名

**Interfaces:**
- Consumes: 当前的文件名列表
- Produces: 符合规范的新文件名

- [ ] **Step 1: 重命名【6. 收集箱】中的乱象笔记**

Run:
```bash
git mv "6. 收集箱/Loop Engineering又是啥？一文讲清企业Agent落地的四层工程进化论.md" "6. 收集箱/企业 Agent 落地四层工程进化论.md"
git mv "6. 收集箱/从玩具到生产力：用真实项目讲透 AI Agent 的 Harness Engineering.md" "6. 收集箱/AI Agent Harness Engineering 实战指南.md"
git mv "6. 收集箱/如果你的 AI Agent 总是失败，请阅读这篇文章.md" "6. 收集箱/AI Agent 常见失败原因与排障指南.md"
git mv "6. 收集箱/JavaGo 开发者 AI 应用开发与 Agent 学习路线（2026 最新版）.md" "6. 收集箱/Java 与 Go 开发者转 AI 应用学习路线.md"
git mv "6. 收集箱/Java-to-AI-roadMap - AI 应用开发与 Agent 学习路线.md" "6. 收集箱/Java 转 AI 应用开发路线图.md"
git mv "6. 收集箱/2026-06-30.md" "6. 收集箱/AI 学习随手记-0630.md"
git mv "6. 收集箱/2026-07-06.md" "6. 收集箱/AI 学习随手记-0706.md"
git mv "6. 收集箱/未命名.md" "6. 收集箱/Agent 开发杂记.md"
git mv "6. 收集箱/JavaGuide 草稿.md" "6. 收集箱/JavaGuide 基础框架梳理草稿.md"
```

- [ ] **Step 2: 重命名【3. 工作与业务】中的超长/口语化笔记**

Run:
```bash
git mv "3. 工作与业务/保险核赔业务/票据风控专题：废票、假票、未查验票、红冲票、重复票、忽略重复、风险提示之间的关系.md" "3. 工作与业务/保险核赔业务/票据风控专题与风险处理手册.md"
git mv "3. 工作与业务/保险核赔业务/资金与结算专题：冻结、解冻、打款、回传、回销、返还票据、挂账、结算状态.md" "3. 工作与业务/保险核赔业务/资金结算专题与状态流转手册.md"
git mv "3. 工作与业务/保险核赔业务/按一个典型真实案件走一遍：从进件到结案.md" "3. 工作与业务/保险核赔业务/真实案件进件至结案全流程履约.md"
git mv "3. 工作与业务/履历与项目经历/sekiro 智能报销审批平台项目经历.md" "3. 工作与业务/履历与项目经历/Sekiro 智能报销审批平台项目经历.md"
```

- [ ] **Step 3: 重命名【2. 实用工具】中的非规范笔记**

Run:
```bash
git mv "2. 实用工具/AI工具手册/claude-code-best-practice.md" "2. 实用工具/AI工具手册/Claude Code 最佳实践指南.md"
git mv "2. 实用工具/AI工具手册/Skills/caveman 安装使用最佳实践.md" "2. 实用工具/AI工具手册/Skills/Caveman Skill 安装使用指南.md"
git mv "2. 实用工具/AI工具手册/Skills/graphify 安装使用最佳实践.md" "2. 实用工具/AI工具手册/Skills/Graphify Skill 安装使用指南.md"
git mv "2. 实用工具/AI工具手册/Skills/README.md" "2. 实用工具/AI工具手册/Skills/Agent Skills 概述与索引.md"
```

- [ ] **Step 4: 重命名【1. 技术研发】中的长说明/疑问句笔记**

Run:
```bash
git mv "1. 技术研发/DevOps与运维/VPS 自建个人 VPN：3X-UI、VLESS-Reality 与 Clash Verge 配置.md" "1. 技术研发/DevOps与运维/VPS 自建 VPN 与 Clash 配置指南.md"
git mv "1. 技术研发/后端与架构/Distributed Design/Spring-State-Machine-分布式落地与局限.md" "1. 技术研发/后端与架构/Distributed Design/Spring StateMachine 分布式落地与局限.md"
git mv "1. 技术研发/后端与架构/Distributed Design/Temporal-io-分布式工作流引擎详解.md" "1. 技术研发/后端与架构/Distributed Design/Temporal 分布式工作流引擎详解.md"
git mv "1. 技术研发/AI应用开发/Java 转 AI 应用开发/02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？.md" "1. 技术研发/AI应用开发/Java 转 AI 应用开发/02-Prompt 与上下文/大模型提示词工程与实用技巧指南.md"
```

---

### Task 2: 重新扫描全库并更新 `wiki/index.md`

**Files:**
- Modify: `wiki/index.md`

**Interfaces:**
- Consumes: 重命名后的新路径列表
- Produces: 最新的 index.md 索引页

- [ ] **Step 1: 运行 Python 脚本生成全新的 `wiki/index.md`**

Run:
```bash
python3 -c '
import os

dirs_to_scan = ["1. 技术研发", "2. 实用工具", "3. 工作与业务", "4. 思考与认知", "5. 项目与作品", "6. 收集箱"]

out = """---
title: "全局笔记索引与清单"
updated: 2026-07-24
---

# 全局笔记索引与清单

"""

for d in dirs_to_scan:
    out += f"## {d}\n\n"
    if os.path.exists(d):
        for r, _, files in os.walk(d):
            rel_dir = r[len(d):].lstrip("/")
            md_files = [f for f in files if f.endswith(".md")]
            if md_files:
                if rel_dir:
                    out += f"### {rel_dir}\n"
                for f in sorted(md_files):
                    path = os.path.join(r, f)
                    name = f[:-3]
                    out += f"- [[{path}|{name}]]\n"
                out += "\n"
    out += "\n"

with open("wiki/index.md", "w", encoding="utf-8") as f:
    f.write(out)
print("wiki/index.md updated successfully.")
'
```

---

### Task 3: Commit Git 节点

**Files:**
- Commit all changes

- [ ] **Step 1: 提交 git 修改**

Run:
```bash
git add .
git commit -m "style: normalize note filenames and update wiki index"
```

---
