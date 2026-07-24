# 死链接与子目录 index.md 清洗 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 清理全库 110 处死链接/无效链接，更新修正因先前重命名导致的旧 Wikilinks，重构 `Java 转 AI 应用开发/index.md` 和 `Hello-Agents/index.md` 消除冗余与路径混乱，并重新生成 `wiki/index.md`。

**Architecture:** 
1. 建立全局新旧文件名映射词典，运行 Python 脚本全库修正旧的 `[[旧文件名]]` 为 `[[新文件名]]`。
2. 清理指向已废弃 `[[.raw/...]]` 和 `[[wiki/learning/...]]` 的破坏性相对路径死链，指向真实存在的笔记。
3. 重构子目录 `index.md`，清除内部无效链接。
4. 重新全量更新 `wiki/index.md` 并提交 git 节点。

**Tech Stack:** Python, Shell, Git.

## Global Constraints

- 绝不改动正文核心知识，仅修正 `[[Wikilinks]]` 中的死链目标与无效相对路径。
- 修改后必须全量扫描确认死链接归零或降至最低。

---

### Task 1: 修正因重命名导致的旧文件名 Wikilinks 死链

**Files:**
- Modify: 全库涉及旧 Wikilinks 的 Markdown 笔记

**Interfaces:**
- Consumes: 新旧文件名映射字典
- Produces: 链接精准可跳转的笔记

- [ ] **Step 1: 运行 Python 脚本修正全库中指向已重命名文件的 Wikilinks**

Run:
```bash
python3 -c '
import os, re

# 新旧文件名映射关系
rename_map = {
    "Spring-State-Machine-分布式落地与局限": "Spring StateMachine 分布式落地与局限",
    "Temporal-io-分布式工作流引擎详解": "Temporal 分布式工作流引擎详解",
    "claude-code-best-practice": "Claude Code 最佳实践指南",
    "caveman 安装使用最佳实践": "Caveman Skill 安装使用指南",
    "graphify 安装使用最佳实践": "Graphify Skill 安装使用指南",
    "sekiro 智能报销审批平台项目经历": "Sekiro 智能报销审批平台项目经历",
    "票据风控专题：废票、假票、未查验票、红冲票、重复票、忽略重复、风险提示之间的关系": "票据风控专题与风险处理手册",
    "资金与结算专题：冻结、解冻、打款、回传、回销、返还票据、挂账、结算状态": "资金结算专题与状态流转手册",
    "按一个典型真实案件走一遍：从进件到结案": "真实案件进件至结案全流程履约",
    "VPS 自建个人 VPN：3X-UI、VLESS-Reality 与 Clash Verge 配置": "VPS 自建 VPN 与 Clash 配置指南",
    "大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？": "大模型提示词工程与实用技巧指南",
    "从玩具到生产力：用真实项目讲透 AI Agent 的 Harness Engineering": "AI Agent Harness Engineering 实战指南",
    "如果你的 AI Agent 总是失败，请阅读这篇文章": "AI Agent 常见失败原因与排障指南",
    "Loop Engineering又是啥？一文讲清企业Agent落地的四层工程进化论": "企业 Agent 落地四层工程进化论",
    "JavaGo 开发者 AI 应用开发与 Agent 学习路线（2026 最新版）": "Java 与 Go 开发者转 AI 应用学习路线",
    "Java-to-AI-roadMap - AI 应用开发与 Agent 学习路线": "Java 转 AI 应用开发路线图"
}

dirs = ["1. 技术研发", "2. 实用工具", "3. 工作与业务", "4. 思考与认知", "5. 项目与作品", "6. 收集箱", "wiki"]

modified_files = 0
for d in dirs:
    if not os.path.exists(d): continue
    for root, _, files in os.walk(d):
        for f in files:
            if f.endswith(".md"):
                path = os.path.join(root, f)
                with open(path, "r", encoding="utf-8", errors="ignore") as file_obj:
                    content = file_obj.read()
                
                new_content = content
                for old_name, new_name in rename_map.items():
                    if old_name in new_content:
                        new_content = new_content.replace(old_name, new_name)
                
                if new_content != content:
                    with open(path, "w", encoding="utf-8") as file_obj:
                        file_obj.write(new_content)
                    modified_files += 1

print(f"Updated old Wikilinks in {modified_files} files.")
'
```

---

### Task 2: 重构/清理子目录 `index.md` 擦除废弃相对路径

**Files:**
- Modify: `1. 技术研发/AI应用开发/Java 转 AI 应用开发/index.md`
- Modify: `1. 技术研发/AI应用开发/Hello-Agents/index.md`

**Interfaces:**
- Consumes: 子目录 index.md
- Produces: 消除死链接后的规范索引页

- [ ] **Step 1: 运行脚本清除 `index.md` 中的废弃死链**

Run:
```bash
python3 -c '
import os, re

files_to_clean = [
    "1. 技术研发/AI应用开发/Java 转 AI 应用开发/index.md",
    "1. 技术研发/AI应用开发/Hello-Agents/index.md"
]

for path in files_to_clean:
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8", errors="ignore") as f:
            content = f.read()
        
        # 清理指向旧 .raw, old wiki, 或者含 ../../../ 相对路径的逻辑链接
        lines = content.splitlines()
        new_lines = []
        for line in lines:
            if ".raw/articles" in line or "wiki/learning/_index" in line or "../../../" in line:
                continue # 跳过过时废弃路径
            new_lines.append(line)
        
        with open(path, "w", encoding="utf-8") as f:
            f.write("\n".join(new_lines) + "\n")

print("Cleaned broken paths in sub-directory index files.")
'
```

---

### Task 3: 重新扫描全库生成 `wiki/index.md` 并 Commit Git 节点

**Files:**
- Modify: `wiki/index.md`

**Interfaces:**
- Consumes: 清洁后的库文件
- Produces: 全最新的 wiki/index.md

- [ ] **Step 1: 重新生成 `wiki/index.md`**

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
'
```

- [ ] **Step 2: 提交 Git 修改**

Run:
```bash
git add .
git commit -m "fix: resolve broken wikilinks and sanitize sub-directory index files"
```

---
