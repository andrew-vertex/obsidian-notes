---
title: "Git 实用技巧：worktree、rebase 与日常分支管理"
created: "2026-07-05"
description: "面向日常开发的 Git 技巧笔记，重点解释 worktree、rebase、stash、cherry-pick、reflog 等高频工具的用途、场景和命令示例。"
tags: [git, dev-tool, workflow, engineering]
tool: "git"
version: ""
platform: mac
layer: 3
status: developing
---

# Git 实用技巧：worktree、rebase 与日常分支管理

> 适用场景：你已经会 `git add` / `git commit` / `git push`，但想更稳地处理多任务并行、分支同步、提交整理、临时切换和误操作恢复。

## 目录

- [Key Takeaways](#key-takeaways)
- [Git 的最小心智模型](#git-的最小心智模型)
- [每次动手前的安全检查](#每次动手前的安全检查)
- [worktree：同一个仓库开多个工作目录](#worktree同一个仓库开多个工作目录)
- [rebase：把一串提交重新接到新基底上](#rebase把一串提交重新接到新基底上)
- [stash：临时搁置未完成改动](#stash临时搁置未完成改动)
- [cherry-pick：只拿某几个提交](#cherry-pick只拿某几个提交)
- [restore / revert / reset：三种回退不要混用](#restore--revert--reset三种回退不要混用)
- [reflog：本地后悔药](#reflog本地后悔药)
- [bisect：二分定位坏提交](#bisect二分定位坏提交)
- [日常推荐工作流](#日常推荐工作流)
- [命令速查](#命令速查)
- [参考来源](#参考来源)

## Key Takeaways

1. **worktree 解决的是“多个分支同时拥有独立目录”的问题**。它不是分支的替代品，而是让你不用 stash、不用来回 `switch`，就能并行处理多个任务。
2. **rebase 解决的是“让当前分支基于最新目标分支继续开发”的问题**。它会重写当前分支提交的 commit hash，所以只适合整理自己的、本地的、未被多人依赖的提交。
3. **merge 保留历史真实分叉，rebase 改写历史让线性阅读更清晰**。团队主干通常追求可追溯，个人 feature 分支通常追求提交干净。
4. **Git 高风险命令的共同原则：先看状态，再看 diff，再执行**。常用三件套是 `git status --short`、`git diff`、`git diff --staged`。
5. **误操作后先别慌着继续操作**。很多本地错误可以用 `git reflog` 找回，但前提是不要连续执行更多破坏性命令。

## Git 的最小心智模型

Git 日常使用可以先理解成 5 个对象：

| 概念 | 直觉理解 | 常用命令 |
|------|----------|----------|
| working tree | 当前目录里的真实文件 | `git status`, `git diff` |
| index / stage | 准备进入下一次 commit 的暂存区 | `git add`, `git diff --staged` |
| commit | 一次不可变快照 | `git commit`, `git show` |
| branch | 指向某个 commit 的可移动指针 | `git switch`, `git branch` |
| remote | 远端仓库的引用 | `git fetch`, `git pull`, `git push` |

常见路径：

```mermaid
flowchart LR
    A[working tree<br/>工作区] -->|git add| B[index<br/>暂存区]
    B -->|git commit| C[local branch<br/>本地分支]
    C -->|git push| D[remote branch<br/>远端分支]
    D -->|git fetch| E[origin/main 等远端引用]
```

一句话：**文件先在工作区变化，`git add` 后进入暂存区，`git commit` 后成为本地历史，`git push` 后同步到远端。**

## 每次动手前的安全检查

任何切分支、rebase、回退、清理之前，先跑：

```bash
git status --short
git branch --show-current
git diff
git diff --staged
```

如果输出里有你没打算处理的改动，先停一下，判断它们属于哪类：

| 状态 | 处理方式 |
|------|----------|
| 自己当前任务的改动 | 继续当前任务 |
| 自己未完成但要切换任务 | `git stash push -u -m "说明"` 或创建 worktree |
| 别人的/不确定的改动 | 不要覆盖，不要回退，先确认来源 |
| 已 staged 但不想提交 | `git restore --staged <file>` |

## worktree：同一个仓库开多个工作目录

### 它是干嘛的？

`git worktree` 允许同一个 Git 仓库同时拥有多个工作目录。

普通情况：

```text
repo/                  # 当前只能 checkout 一个 branch
```

worktree 情况：

```text
repo/                  # main 分支
.worktrees/fix-bug/    # fix/bug 分支
.worktrees/feature-a/  # feature/a 分支
```

这些目录共享同一个 Git 对象库，但每个 worktree 都有自己的：

- checked out branch
- working tree 文件
- staged changes
- uncommitted changes

所以你可以一边保留当前目录的未完成改动，一边在另一个目录处理紧急 bug。

### 什么时候适合用？

| 场景 | 是否推荐 | 原因 |
|------|----------|------|
| 当前分支有未完成改动，突然要修线上 bug | 推荐 | 避免 stash 和切分支污染现场 |
| 同时做两个互不相关 feature | 推荐 | 每个任务有独立目录、独立分支 |
| 让 AI agent 做一个风险较大的改动 | 推荐 | 降低误动当前工作区的概率 |
| 对比两个分支运行效果 | 推荐 | 两边可以同时开服务 |
| 当前工作区干净，只改一个小文件 | 不一定 | 直接新建分支更轻 |
| 项目依赖安装很重 | 谨慎 | 多个目录可能重复占用构建缓存或 `node_modules` |

### 基本使用

查看当前已有 worktree：

```bash
git worktree list
```

创建新 worktree，并新建分支：

```bash
git worktree add .worktrees/fix-0098 -b fix/0098-bst-validation
cd .worktrees/fix-0098
git status
```

在新目录正常开发、提交：

```bash
git add docs/problems/leetcode-98-validate-binary-search-tree.md
git commit -m "docs: refine BST validation note"
```

从已有分支创建 worktree：

```bash
git worktree add .worktrees/fix-0098 fix/0098-bst-validation
```

### 重要限制

同一个 branch 不能同时被两个 worktree checkout。

例如 `main` 已经在主目录，就不能再直接：

```bash
git worktree add .worktrees/main-copy main
```

如果只是想看某个 commit，可以用 detached HEAD；如果要继续开发，应该新建分支。

### 清理 worktree

先确认 worktree 干净：

```bash
cd .worktrees/fix-0098
git status --short
```

再回主仓库执行：

```bash
cd ../..
git worktree remove .worktrees/fix-0098
git worktree prune
```

> [!warning] 本 vault 的本地规则
> 在这个知识库里，删除文件/目录优先用 `trash` 进入 macOS 回收站。实际清理前先 `git status --short` 和 `ls` 确认路径。

## rebase：把一串提交重新接到新基底上

### 它是干嘛的？

`rebase` 的字面意思是“重新设置 base”。

假设你从 `main` 拉出 `feature/login` 后，别人又往 `main` 合了两个提交：

```text
A---B---C  main
     \
      D---E  feature/login
```

你在 `feature/login` 执行：

```bash
git fetch origin
git rebase origin/main
```

结果变成：

```text
A---B---C  origin/main
         \
          D'---E'  feature/login
```

注意：`D'`、`E'` 不是原来的 `D`、`E`。它们内容可能一样，但 commit hash 变了，因为 Git 重新创建了提交。

### 为什么 rebase 能让历史线性化？

因为它不是创建一个 merge commit，而是把你的提交一个个“重放”到新的基底上。

对比：

```bash
git merge origin/main
```

会保留分叉和合并：

```text
A---B---C------M  feature/login
     \        /
      D---E---
```

而：

```bash
git rebase origin/main
```

会让 feature 看起来像是从最新 `main` 上开始开发：

```text
A---B---C---D'---E'  feature/login
```

这就是“线性化”的来源。

### 什么时候用 rebase？

| 场景 | 推荐做法 |
|------|----------|
| 自己的 feature 分支落后 `main`，想基于最新代码继续开发 | `git fetch origin && git rebase origin/main` |
| 提 PR 前想整理自己的提交 | `git rebase -i origin/main` |
| 分支已经被多人共同开发、别人也基于它继续提交 | 谨慎，通常用 merge |
| 已经 push 到远端且被别人拉走 | 不要随便 rebase，除非团队明确允许 |

### 常用 rebase 命令

把当前分支接到最新 `main` 上：

```bash
git fetch origin
git rebase origin/main
```

交互式整理最近 3 个提交：

```bash
git rebase -i HEAD~3
```

常见交互动作：

| 动作 | 含义 |
|------|------|
| `pick` | 保留这个提交 |
| `reword` | 保留提交内容，但修改 commit message |
| `squash` | 合并到前一个提交，并合并 message |
| `fixup` | 合并到前一个提交，丢弃当前 message |
| `drop` | 删除这个提交 |

遇到冲突时：

```bash
# 1. 手动解决冲突文件
git status
git add <resolved-file>

# 2. 继续 rebase
git rebase --continue
```

放弃本次 rebase：

```bash
git rebase --abort
```

### rebase 后如何 push？

如果 rebase 的分支已经 push 过，commit hash 会变化，普通 push 可能失败。

更安全的方式是：

```bash
git push --force-with-lease
```

`--force-with-lease` 比 `--force` 更安全：如果远端在你没注意时又有别人推了新提交，它会拒绝覆盖。

> [!warning] 团队协作原则
> 只 rebase 自己负责、别人没有基于其继续开发的分支。公共主干、测试分支、发布分支不要随意 rebase。

## stash：临时搁置未完成改动

`stash` 适合“我现在有未提交改动，但想临时切走一下”。

保存当前改动，包括未跟踪文件：

```bash
git stash push -u -m "wip: claim review note"
```

查看 stash：

```bash
git stash list
```

恢复最近一次 stash，并从 stash 列表删除：

```bash
git stash pop
```

只恢复但保留 stash 记录：

```bash
git stash apply stash@{0}
```

适合用 stash 的情况：

- 临时切分支处理小事。
- 需要 pull/rebase 前先清空工作区。
- 改动很小，还没必要 commit。

不适合长期用 stash 的情况：

- 改动已经持续几天。
- stash 里有多个上下文混在一起。
- 任务已经有明确方向。

这种情况更建议开分支或 worktree。

## cherry-pick：只拿某几个提交

`cherry-pick` 是把别的分支上的某个 commit 复制到当前分支。

典型场景：

- 一个 bug fix 已经在 feature 分支上，需要先带到 release 分支。
- 测试分支只想发布某个明确 commit，不想 merge 整个 feature。
- 从别人分支里挑一个独立提交。

示例：

```bash
git switch release/2026-07
git cherry-pick abc1234
```

连续拿多个提交：

```bash
git cherry-pick abc1234 def5678
```

拿一个范围：

```bash
git cherry-pick A^..B
```

冲突处理：

```bash
git status
git add <resolved-file>
git cherry-pick --continue
```

放弃：

```bash
git cherry-pick --abort
```

## restore / revert / reset：三种回退不要混用

| 命令 | 主要作用 | 是否改历史 | 日常安全性 |
|------|----------|------------|------------|
| `git restore` | 恢复工作区或取消暂存 | 不改 commit 历史 | 相对安全 |
| `git revert` | 新增一个“反向提交”来撤销旧提交 | 不改旧历史 | 团队协作安全 |
| `git reset` | 移动当前分支指针，可影响暂存区和工作区 | 会改历史 | 高风险 |

### restore：撤销文件级改动

取消暂存：

```bash
git restore --staged <file>
```

丢弃工作区某个文件的未提交改动：

```bash
git restore <file>
```

> [!warning] `git restore <file>` 会丢掉该文件未提交改动。执行前先 `git diff <file>`。

### revert：团队协作里推荐的撤销提交

撤销某个已经提交、甚至已经 push 的 commit：

```bash
git revert abc1234
```

它会产生一个新的 commit，不会改写历史，因此适合公共分支。

### reset：只在你非常确定时使用

常见但危险的形式：

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

区别：

| 模式 | branch 指针 | 暂存区 | 工作区 |
|------|-------------|--------|--------|
| `--soft` | 回退 | 保留 | 保留 |
| `--mixed` | 回退 | 清空 | 保留 |
| `--hard` | 回退 | 清空 | 清空 |

> [!warning] 尽量避免 `reset --hard`
> 它会直接丢弃工作区改动。除非你已经确认所有重要内容都已提交、备份或不再需要。

## reflog：本地后悔药

`reflog` 记录本地 HEAD 和分支指针移动历史。

误 rebase、误 reset 后，先看：

```bash
git reflog
```

你会看到类似：

```text
abc1234 HEAD@{0}: rebase finished
def5678 HEAD@{1}: checkout: moving from main to feature/login
789abcd HEAD@{2}: commit: add login validation
```

如果想回到某个位置，可以先新建一个救援分支：

```bash
git switch -c rescue-before-bad-rebase HEAD@{2}
```

建议先建救援分支，而不是马上 reset 回去。这样保留现场，方便对比。

## bisect：二分定位坏提交

当你知道“现在坏了，某个旧版本是好的”，但不知道中间哪个 commit 引入问题时，用 `git bisect`。

```bash
git bisect start
git bisect bad
git bisect good v1.2.0
```

Git 会自动 checkout 中间提交。你运行测试后告诉它结果：

```bash
git bisect good
# 或
git bisect bad
```

结束后：

```bash
git bisect reset
```

适合定位：

- 某个接口从什么时候开始失败。
- 某个性能退化从哪个提交引入。
- 某个测试在一串提交中突然坏掉。

## 日常推荐工作流

### 1. 开始一个普通 feature

```bash
git switch main
git pull --ff-only
git switch -c feature/claim-review-note
```

开发过程中小步提交：

```bash
git status --short
git diff
git add <files>
git commit -m "docs: add claim review git note"
```

### 2. 当前目录有未完成改动，但要紧急修 bug

优先考虑 worktree：

```bash
git worktree add .worktrees/fix-urgent-claim-bug -b fix/urgent-claim-bug
cd .worktrees/fix-urgent-claim-bug
```

修完后：

```bash
git status --short
git add <files>
git commit -m "fix: handle urgent claim edge case"
```

### 3. feature 开发几天后，同步 main 最新代码

```bash
git fetch origin
git rebase origin/main
```

冲突解决完：

```bash
git rebase --continue
```

如果这个 feature 已经 push 过：

```bash
git push --force-with-lease
```

### 4. 提 PR 前整理提交

```bash
git fetch origin
git rebase -i origin/main
```

常见整理方式：

- 多个“修 typo / fix lint / update”合并成一个有意义的提交。
- 保留能独立 review 的提交边界。
- commit message 改成团队能读懂的说明。

### 5. 公共分支撤销错误提交

不要 rebase 公共分支，也不要随便 reset。用：

```bash
git revert <bad-commit>
git push
```

## 命令速查

| 命令 | 作用 | 示例 |
|------|------|------|
| `git status --short` | 查看简洁状态 | `git status --short` |
| `git diff` | 看工作区未暂存改动 | `git diff` |
| `git diff --staged` | 看已暂存改动 | `git diff --staged` |
| `git switch -c <branch>` | 新建并切换分支 | `git switch -c feature/login` |
| `git pull --ff-only` | 只允许快进拉取 | `git pull --ff-only` |
| `git worktree list` | 查看 worktree | `git worktree list` |
| `git worktree add <path> -b <branch>` | 新建 worktree 和分支 | `git worktree add .worktrees/fix-a -b fix/a` |
| `git worktree remove <path>` | 移除 worktree | `git worktree remove .worktrees/fix-a` |
| `git rebase <base>` | 把当前分支重放到 base 上 | `git rebase origin/main` |
| `git rebase -i <base>` | 交互式整理提交 | `git rebase -i HEAD~3` |
| `git rebase --abort` | 放弃 rebase | `git rebase --abort` |
| `git push --force-with-lease` | 安全一些的强推 | `git push --force-with-lease` |
| `git stash push -u -m "<msg>"` | 暂存当前改动 | `git stash push -u -m "wip"` |
| `git stash pop` | 恢复最近 stash | `git stash pop` |
| `git cherry-pick <commit>` | 拿一个提交到当前分支 | `git cherry-pick abc1234` |
| `git restore --staged <file>` | 取消暂存 | `git restore --staged app.py` |
| `git restore <file>` | 丢弃文件工作区改动 | `git restore app.py` |
| `git revert <commit>` | 用新提交撤销旧提交 | `git revert abc1234` |
| `git reflog` | 查看 HEAD 移动历史 | `git reflog` |
| `git bisect start` | 开始二分定位 | `git bisect start` |

## 常见问题

> [!question] 我平时到底应该用 merge 还是 rebase？
> 个人 feature 分支同步主干、整理提交时，优先 rebase；公共分支、多人共享分支、发布分支，优先 merge 或 revert。

> [!question] rebase 会不会丢代码？
> 正常不会，但它会重写提交。如果冲突解决错、再强推，就可能覆盖远端历史。执行前先 `git status` / `git log --oneline --graph --decorate -20`，出事后先看 `git reflog`。

> [!question] worktree 和 stash 怎么选？
> 小而短的临时切换用 stash；独立任务、可能持续一段时间、需要跑服务或交给 AI agent 的改动，用 worktree。

> [!question] 为什么不直接一直在 main 上改？
> main 应该保持可运行、可回滚、可发布。feature 分支让每个任务有清晰边界，也方便 review 和撤销。

## 关联

- [[learning/_index|学习 MOC]] — 技术工具与工程工作流入口
- [[Tools/AI/Superpowers 安装使用与 AI 编程工作流实践指南]] — AI 编程工作流中也依赖 worktree 做隔离

## 参考来源

- 本次对话：用户提出的 Git worktree / rebase 学习需求与示例说明。
- 本地 Superpowers skill：`using-git-worktrees`，用于校准 worktree 的隔离场景和安全原则。
- Git 官方文档：<https://git-scm.com/docs/git-worktree>
- Git 官方文档：<https://git-scm.com/docs/git-rebase>
- Git 官方文档：<https://git-scm.com/docs/git-stash>
- Git 官方文档：<https://git-scm.com/docs/git-cherry-pick>
- Git 官方文档：<https://git-scm.com/docs/git-reflog>
