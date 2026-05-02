---
title: "If your AI agent keeps failing, read this"
source: "https://strategizeyourcareer.com/p/harness-engineering-ai-agents"
author:
  - "[[Fran Soto]]"
published: 2025-04-27
created: 2026-04-28
description: "What is harness engineering? Learn how to turn unreliable AI coding agents into production systems with state management, guardrails, and deterministic scripts. Real examples from Amazon"
tags:
  - "clippings"
---
### Learn how harness engineering uses deterministic scripts, guardrails, and state management to turn AI coding agents into production systems that ship real code.学习束流工程师如何利用确定性脚本、护栏和状态管理，将 AI 编码代理转变为生产系统，交付真实代码。

Most AI coding agents can write impressive demos. Few can ship production code without breaking everything around it. The difference is harness engineering: the discipline of building systems that make AI agents reliable.  
大多数 AI 编码代理都能写出令人印象深刻的演示。很少有人能在不破坏周围一切的情况下发布生产代码。区别在于机束工程：构建使人工智能智能体可靠的系统。

Here is how I used it to ship 100+ PRs/month at Amazon  
这是我用它在亚马逊每月发送100+永久居留币的方式

---

Get the free AI Agent Building Blocks ebook when you subscribe:  
订阅后即可免费获取《人工智能代理人构建模块》电子书：

![Ebook cover of "AI Agents Building Blocks"](https://substackcdn.com/image/fetch/$s_!p39U!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1f389e59-901f-4e99-b38c-56bf136001a8_1628x2624.png)

Ebook cover of "AI Agents Building Blocks"

---

I’m Fran. I’m a software engineer at Amazon during the day, and I write and experiment with AI during the night.  
我是弗兰。我白天在亚马逊做软件工程师，晚上写作和实验人工智能。

I want to tell you about the moment I realized that prompting alone would never work for production AI agents.  
我想告诉你我意识到仅靠提示对生产 AI 代理永远行不通的那一刻。

I was working on an automation project at Amazon. The goal was simple: update large JSON configuration files automatically based on requirements. These configs were thousands of lines long, and the updates followed predictable patterns.  
我当时在亚马逊做一个自动化项目。目标很简单：根据需求自动更新大型 JSON 配置文件。这些配置有成千上万行，更新模式也都很可预测。

A perfect job for an AI agent, right? That’s what everyone thought. Engineers on the team opened their AI-powered IDE or CLI, typed their prompts to modify the JSONs, and watched the LLM struggle to modify the target node correctly.  
这不是 AI 代理的完美工作吗？大家都这么想。团队中的工程师打开了由 AI 驱动的 IDE 或 CLI，输入修改 JSON 的提示，并观看 LLM 努力正确修改目标节点。

It failed to implement the changes properly. Every single time.  
但未能正确实施这些变更。每一次都是这样。

The model wasn’t broken. We were on Opus 4.6 with a one-million context window.  
这个模式并没有被破坏。我们当时用的是 Opus 4.6，拥有一百万个上下文窗口。

The context window was a problem. When you feed multiple 10,000-line JSON files into an LLM, the model loses track of the surrounding structure. It edits what you asked it to edit, but it quietly breaks everything around it. No error message. No warning. Just a structurally invalid file that passes a surface-level glance but fails in production.  
上下文窗口是个问题。当你向一个大型语言模型输入多个 1 万行的 JSON 文件时，模型会失去对周围结构的跟踪。它会编辑你让它修改的内容，但悄悄地破坏周围的一切。没有错误提示。没有任何预警。只是一个结构无效的文件，表面上能被浏览，但在生产环境中失败。

This is not a model quality problem. It is an **environment** problem. And the fix is not a better prompt.  
这不是模型质量的问题。这是 **环境** 问题。而修正并不是更好的提示。

You may think the fix is Anthropic to release a 10M context window, but we know that a bigger context window still degrades after 100k or 200k tokens.  
你可能认为修复方法是 Anthropic 发布 1000 万上下文窗口，但我们知道更大的上下文窗口在 10 万或 20 万代币后仍然会下降。

The real fix is a **harness**.  
真正的解决办法是背 **带** 。

Harness engineering is the discipline that turned my broken prototype at Amazon into a system that now ships over 100 PRs per month. Fully autonomous.  
线束工程正是把我在亚马逊那个坏掉的原型机变成现在每月出货超过 100 个 PR 的系统。完全自主。

I wrote a 10-step guide to build that agent in this previous post:  
我在之前的文章中写过一篇构建该代理的10步指南：

---

## In this post, you’ll learn在这篇文章中，你将学到东西

- What harness engineering is and how it differs from prompt engineering, context engineering, and agent engineering  
	什么是线束工程，以及它与提示工程、上下文工程和代理工程的区别
- Why AI agents fail on large structured files like JSON, and how to fix it with deterministic scripts  
	为什么 AI 代理在大型结构文件如 JSON 上失败，以及如何用确定性脚本修复这个问题
- The four pillars of a production AI harness: state management, context architecture, guardrails, and entropy management  
	生产型 AI 的四大支柱：状态管理、上下文架构、护栏和熵管理
- How I built a harness at Amazon that ships 100+ PRs/month without human intervention  
	我在亚马逊打造了一个背带，每月能发 100+个 PR 且无需人工干预
- The mindset shift that separates engineers who demo AI from engineers who deploy it  
	这种思维转变区分了演示 AI 的工程师与部署 AI 的工程师

---

## Why AI Agents Fail on Large Files为什么 AI 代理在处理大文件时会失败

Most engineers today interact with AI coding tools the same way: open Cursor, Claude Code, or Codex, type a prompt, review the output, repeat. For small files and isolated tasks, this works beautifully. But the moment the problem involves a large amount of files, the whole approach falls apart.  
如今大多数工程师与 AI 编码工具的交互方式相同：打开光标、Claude 代码或 Codex，输入提示词，审核输出，重复。对于小文件和独立任务，这种方法非常有效。但一旦问题涉及大量文件，整个方法就会崩溃。

![A stick-figure robot looking with a magnifier (inside the context window). Inside the magnifier, everything is tidy. Outside it's a mess. ](https://substackcdn.com/image/fetch/$s_!you5!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe26f2f78-9ef9-4c74-90df-29706194a08a_1476x899.png)

A stick-figure robot looking with a magnifier (inside the context window). Inside the magnifier, everything is tidy. Outside it's a mess.

Large Language Models are probabilistic engines. They predict the next token based on patterns in their context window. When the context window is filled with thousands of lines of structured data, the model’s attention gets diluted. It correctly identifies the node you want to modify, but it loses track of sibling keys, nested brackets, and structural integrity. The result is a file that looks right at the point of change but is broken somewhere else.  
大型语言模型是概率引擎。他们根据上下文窗口中的模式预测下一个代币。当上下文窗口被成千上万行结构化数据填满时，模型的注意力会被稀释。它能正确识别你想修改的节点，但会丢失兄弟键、嵌套括号和结构完整性的跟踪。结果是文件在变化点看起来正确，但其他地方却坏了。

We have to understand that **Context Window** isn’t the same as **Context Attention.** As a human, I can store hundreds of items in a storage unit, but I will remember about a fraction of the items I have there.  
我们必须明白 ， **情境窗口** 和情境注意力不是同一回 **事。** 作为人类，我可以在储物单元里存放数百件物品，但我记得的只有其中一小部分 。

Same with LLMs. Performance degrades as the context window gets filled (and costs).  
大型语言模型也是如此。随着上下文窗口被填满（成本增加），性能会下降。

> **Did you know that every message you send is sending all the previous conversation in an API call?** Yes, you’re billed also for those past messages. The servers in the cloud don’t keep any state, they only have a cache.  
> **你知道吗，你发送的每条消息都是在 API 调用中发送之前所有的对话内容？** 是的，你也要为那些过去的留言收费。云端的服务器不保留任何状态，它们只有缓存。

![A comparison diagram. On the left it's what you see, a second user question and a second LLM answer. On the right, there's what AI sees: A system prompt, the MCP tool definitions, the user system prompt, the first question, all files referenced, first llm answer, second user question, all files referenced, second llm answer](https://substackcdn.com/image/fetch/$s_!nJAD!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F48304c89-9d25-4dd7-a56f-9b81e6069fbd_1041x1319.png)

A comparison diagram. On the left it's what you see, a second user question and a second LLM answer. On the right, there's what AI sees: A system prompt, the MCP tool definitions, the user system prompt, the first question, all files referenced, first llm answer, second user question, all files referenced, second llm answer

When the model fails to make an update, the instinct is to write a better prompt.  
当模型无法进行更新时，本能反应是写出更好的提示。

Add more constraints. 增加更多约束。

Tell the model to “preserve the surrounding structure.”  
告诉模型“保留周围结构”。

“Make no mistakes.” “别出错。”

But that is like asking someone to juggle while blindfolded and then giving them more detailed instructions about hand positioning. The problem is not the instructions. The problem is the blindfold.  
但这就像让别人蒙眼玩杂耍，然后再给他们更详细的手部摆位指导一样。问题不在于说明书。问题出在蒙眼。

The context window itself becomes a liability when it’s packed with thousands of lines of repetitive structure. No prompt can fix that.  
当上下文窗口被成千上万条重复结构堆积时，反而成了负担。没有任何提示词能解决这个问题。

I covered in this post how to scale AI setting up guardrails  
我在这篇文章中介绍了如何扩展 AI 设置护栏

---

## What Is Harness Engineering?什么是束带工程？

**Harness engineering is the discipline of designing the systems, architectural constraints, execution environments, and automated feedback loops that wrap around AI agents to make them reliable in production.  
利用束工程是设计系统、架构约束、执行环境和自动化反馈回路的学科，这些环绕着人工智能代理，使其在生产环境中更为可靠。**

The term was first coined by Mitchell Hashimoto, the founder of HashiCorp. The metaphor comes from horse riding. Think of the LLM as a powerful horse. It has raw energy, speed, and strength. But without reins, a saddle, and a bridle, that energy is undirected and potentially destructive (the horse kicks you, the LLM runs a `rm -rf`, and I don’t know which is worse`)`. The harness allows the rider to direct the horse’s power productively.  
该术语最早由 HashiCorp 创始人 Mitchell Hashimoto 提出。这个比喻来自骑马。把大型语言模型看作一匹强大的战马。它拥有原始的能量、速度和力量。但没有缰绳、鞍具和缰绳，这种能量是无定向且可能破坏性的（马踢你，LLM 跑 rm `-rf` ，我不知道哪个更糟 `）。` 背带使骑手能够有效地引导马匹的力量。

To understand where harness engineering fits, here’s how it relates to the other disciplines you’ve probably heard about:  
为了理解束缚工程的定位，以下是它与你可能听说过的其他学科的关系：

- **Prompt Engineering** → Single interaction to craft the best input to the model (single request-response interaction).  
	**提示工程** → 单一交互以打造最佳输入（单请求-响应交互）。
- **Context Engineering** → Control what the model sees during a whole session (multiple interactions until clearing).  
	**上下文工程** → 控制模型在整个会话中看到的内容（多次交互直到清除）。
- **Harness Engineering** → Designs the environment, tools, guardrails, and feedback loops (multiple sessions).  
	**利用工程** →设计环境、工具、护栏和反馈循环（多次会议）。
- **Agent Engineering** → Design the agent’s internal reasoning loop (define specialized agents).  
	**代理工程** →设计代理的内部推理循环（定义专用代理）。
- **Platform Engineering** → Infrastructure to manage deployment, scaling, and cloud operations (where agents can run).  
	平台 **工程** →基础设施，用于管理部署、扩展和云运营（代理可运行）。

![A sequence diagram with user, the IDE/CLI client and the backend. The user sending a prompt is what prompt engineering covers. The files and context that IDe/cli AI client sends to backend is context engineering. And all the conversation, including the tools accessible for the IDE/CLI that determine how the backend responds are harness engineering](https://substackcdn.com/image/fetch/$s_!QsJP!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2c1b6df5-e49a-42e4-892c-b8c58a511cae_1240x1047.png)

A sequence diagram with user, the IDE/CLI client and the backend. The user sending a prompt is what prompt engineering covers. The files and context that IDe/cli AI client sends to backend is context engineering. And all the conversation, including the tools accessible for the IDE/CLI that determine how the backend responds are harness engineering

Prompt engineering is about what you say to the model.  
提示工程就是你对模型说什么。

Context engineering is about what the model sees.  
上下文工程关注模型所看到的。

Harness engineering is about the entire world the model operates in. It includes the tools the agent can call, the constraints it cannot violate, the documentation structure it reads, and the automated feedback loops that catch its mistakes before they reach production.  
线束工程涉及模型运行的整个世界。它包括代理可以调用的工具、无法违反的约束、阅读的文档结构，以及自动反馈循环，在错误进入生产环境前及时发现。

---

## How I Built a Harness That Ships 100+ PRs/Month at Amazon我如何打造一个每月能在亚马逊发货 100+ PR 的背带

Let me walk you through the specific problem I solved, because abstract talk about agents only becomes useful when you see them applied to a real constraint.  
让我带你了解我解决的具体问题，因为抽象的智能体讨论只有在你看到它们应用于真实约束时才有用。

**The problem:** We had large JSON configuration files that needed automated, repetitive updates. These files were too big for the LLM’s context window. Every manual update was tedious, error-prone, and time-consuming.  
**问题是：** 我们有大型 JSON 配置文件，需要自动化且重复的更新。这些文件对 LLM 的上下文窗口来说太大了。每次手动更新都既繁琐又容易出错，耗时又长。

**What everyone else tried:** Engineers on the team opened their IDEs and started prompting. The LLM would correctly modify the target node, but would fail to identify which other files had to be updated, and it would fail to keep the correct JSON structure. There was no awareness of JSON structural integrity as a hard constraint. Every run was a coin flip. Sometimes it worked. Most times it broke. You can’t trust an AI like this.  
**其他人尝试的：** 团队中的工程师打开他们的 IDE 并开始提示。LLM 会正确修改目标节点，但无法识别哪些文件需要更新，也无法保持正确的 JSON 结构。当时没有意识到 JSON 的结构完整性是硬性约束。每一次跑动都像掷硬币一样。有时候确实有效。大多数时候它会坏。你不能信任这样的人工智能。

**The harness approach:** Instead of trying to update the prompt, I narrowed the problem to one specific operation: How to read and write into our JSON files. I wasn’t trying to build a general-purpose agent. I built a scoped one. I wrote deterministic Python scripts to handle the actual JSON surgery: read the file, apply a precise modification, validate the structure, write it back. The agent’s only job was to provide the **intent**, the what, and the where. The script provided the **execution guarantee**.  
**利用方法：** 我没有尝试更新提示词，而是将问题范围缩小到一个具体操作：如何读取和写入我们的 JSON 文件。我并不是想打造一个通用代理。我做了一个带瞄准镜的。我写了确定性 Python 脚本来处理实际的 JSON 操作：读取文件，进行精确修改，验证结构，然后写回。代理人的唯一职责就是提供 **意图** 、内容和地点。脚本提供了 **执行保证** 。

The key insight was this: the agent calls the script as a tool. It does not generate JSON directly. It tells the script what to change, and the script changes it with zero ambiguity. This means the AI is the brain that chooses which steps to take, like a CEO indicating directions. The AI didn’t have to make the groundwork itself.  
关键的见解是：代理调用脚本作为工具。它不会直接生成 JSON。它告诉剧本该改什么，剧本也毫无歧义地修改。这意味着人工智能是决定采取哪些步骤的大脑，就像 CEO 指明方向一样。AI 不必自己打基础。

I then added a structural validation step as a guardrail. If the resulting JSON is malformed, the agent cannot proceed. It physically cannot ship a broken config. This provides a feedback loop, which is something managers and C-level executives also want when delegating to humans.  
然后我加了一个结构验证步骤作为护栏。如果得到的 JSON 格式错误，代理无法继续。它物理上无法发布损坏的配置。这形成了一个反馈循环，这也是管理者和高管在委派给人类时所需要的。

**The result:** 100+ PRs per month. Zero structural corruption. Fully autonomous. The system has been running for months, and after a few weeks of tweaking edge cases in the deterministic scripts, the Agent nails the updates.  
**结果是：** 每月 100+个永久冠军。零结构性腐败。完全自主。系统已经运行了数月，经过几周调整确定性脚本中的边缘情况后，特工成功完成了更新。

![Left: A horse going crazy with the title "claude, make no mistakes". Right, a horse with a stick figure on top, titled "AI Engineering Harness"](https://substackcdn.com/image/fetch/$s_!xpdI!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F25e8dd1c-ce22-4030-9ddd-e64ee43b3acf_1595x972.png)

Left: A horse going crazy with the title "claude, make no mistakes". Right, a horse with a stick figure on top, titled "AI Engineering Harness"

At some point, we realized the only reason a PR gets rejected is that the requirement was wrong, not because the AI didn’t execute the requirement.  
后来我们意识到，PR 被拒绝的唯一原因是需求本身错误，而不是因为 AI 没有执行。

That’s when you are into something good.  
那时你正沉浸在美好事物中。

This is what harness engineering looks like in practice. You stop asking the model to do things it’s bad at. You give it the tools for the parts that require precision, you let the agent handle the parts that require judgment, and you instruct it not to jump in to do the job itself.  
这就是线束工程的实际表现。你不再让模特做它不擅长的事情。你给它需要精准操作的工具，让代理人处理需要判断的部分，并指示它不要直接干活。

---

## The Four Pillars of Harness Engineering线束工程的四大支柱

My JSON automation project taught me the pattern to build a good AI agent, but the approach is generic. After studying how OpenAI, Anthropic, and other teams have built their own harnesses, I’ve identified four pillars that every production harness needs.  
我的 JSON 自动化项目教会了我构建优秀 AI 代理的模式，但方法比较通用。在研究了 OpenAI、Anthropic 及其他团队如何构建自己的线束后，我确定了每个生产线束所需的四大支柱。

### 1\. State Management 1. 国家管理

AI agents are stateless by default. Every API call starts with a blank slate. For a task that takes five minutes, this is fine. For a task that spans hours or requires following the updates of dozens of files, statelessness is bad. The agent forgets what it did 20 steps ago. It repeats the same mistake in a loop. It loses track of the overall architecture. This “AI amnesia” is the most common failure mode in long-running agent tasks, and it’s why Openclaw got very popular.  
AI 代理默认是无状态的。每个 API 调用都是从一张白纸开始的。对于一个只需五分钟的任务来说，这已经足够了。对于需要数小时或需要跟踪数十个文件更新的任务来说，无状态状态是糟糕的。客服忘记了 20 步前做了什么。它会在循环中重复同样的错误。它失去了对整体建筑的把握。这种“AI 失忆”是长期运行代理任务中最常见的失败模式，这也是 Openclaw 如此受欢迎的原因。

A harness solves this by serializing context snapshots and restoring them across sessions. Think of it as save points in a video game. The agent does work, the harness saves a snapshot, and if the agent crashes or hits a rate limit, the harness restores the snapshot and picks up exactly where it left off.  
线束通过序列化上下文快照并在会话间恢复来解决这个问题。可以把它想象成电子游戏中的存档点。代理确实工作，线束保存快照，如果代理崩溃或达到速率限制，线束会恢复快照并从中断处恢复。

Advanced implementations use structured state objects that persist across runs. There are two main strategies here:  
高级实现使用跨运行持续存在的结构化状态对象。这里主要有两种策略：

- **Context Compaction**, where the harness continuously summarizes the agent’s history as it approaches the token limit  
	**上下文压缩** ，即当代理接近令牌限制时，束会持续总结代理的历史
- **Context Resets**, where the harness clears the window entirely and boots a fresh agent with a structured handoff of artifacts.  
	上下文 **重置** ，安全带完全清空窗口，启动一个新特工，并有结构地交接神器。

Both work. The right choice depends on your task length and coherence requirements.  
两者都有效。正确的选择取决于你的任务长度和连贯性要求。

### 2\. Context Architecture (Progressive Disclosure)2. 上下文架构（渐进披露）

The first agent-friendly codebases I saw produced gigantic `AGENTS.md` files. This approach fails for the same reason a 500-page employee handbook fails on someone’s first day. The agent gets confused, misses critical rules, and follows outdated instructions that were never cleaned up.  
我见过的第一批代理友好代码库产生了巨大的 `AGENTS.md` 文件。这种方法失败的原因，就像一本 500 页的员工手册在某人第一天上班时失败一样。客服人员会搞混，错过关键规则，还会遵循过时的指令，这些指令从未被清理过来。

The better approach is **progressive disclosure**. Give the agent a short table of contents that points to a structured `docs/` directory. The agent reads the table of contents first, then navigates to the specific document it needs for the task at hand.  
更好的做法是 **渐进式披露** 。给代理一个简短的目录，指向一个结构化 `的文档/` 目录。代理先阅读目录，然后导航到当前任务所需的具体文档。

This is the same pattern introduced with the Agent Skills standard. Instead of the early MCP implementations that loaded all the definitions above the user’s first prompt, let the agent find them when needed.  
这也是特工技能标准引入的模式。与早期 MCP 实现中加载用户第一个提示上所有定义不同，不如让代理在需要时自行查找。

The agent gets a map, not an encyclopedia.  
特工拿的是地图，而不是百科全书。

One more thing that is easy to forget: **anything the agent cannot access in-context does not exist for it.** Your Slack threads, Google Docs, and verbal agreements in meetings… None of that is real to the agent unless provided or instructed to fetch them.  
还有一点容易被遗忘： **代理无法在上下文中访问的任何内容对它来说都不存在。** 你的 Slack 帖子、谷歌文档和会议上的口头协议......除非被提供或指示去取，否则这些都不真实。

### 3\. Deterministic Guardrails3. 确定性护栏

This is where harness engineering diverges most sharply from prompt engineering. Prompt engineering **asks** the agent to write clean code or make no mistakes. Harness engineering **mechanically enforces** it.  
这正是线束工程与即时工程最大区别的地方。提示工程 **要求** 代理写出干净的代码或不犯错。线束工程 **机械上强制执行** 这一点。

You’d need custom linters, structural tests, and CI jobs that validate architecture before merge.  
你需要自定义的 linter、结构测试和 CI 作业，在合并前验证架构。

The agent isn’t “discouraged” or “instructed against” skipping those. The agent is blocked.  
客服不会被“劝阻”或“指示”跳过这些。代理被屏蔽了。

- If a file exceeds a size limit, the linter rejects it.  
	如果文件超过了大小限制，衬板会拒绝该文件。
- If a dependency flows in the wrong direction, the structural test fails.  
	如果依赖方向错误，结构测试失败。
- If the JSON output is malformed, the validation script prevents merging the PR.  
	如果 JSON 输出格式错误，验证脚本会阻止合并 PR。

The error messages in your custom lints and validations should include remediation instructions. When the agent hits a linter failure, the error message itself tells the agent exactly how to fix the problem. That error message gets injected directly into the agent’s context, creating a tight feedback loop that requires zero human intervention.  
自定义 lints 和验证中的错误信息应包含修复说明。当代理遇到衬垫故障时，错误信息本身会准确告诉代理如何修复问题。该错误信息直接注入代理的上下文中，形成一个紧密的反馈循环，完全不需要人工干预。

This was a realization of my early attempts in the agent that modifies JSONs. I was using JQ commands instead of Python scripts. JQ ended all possible failures with a 0 or 1 exit code. These outputs are intended for terminals, not for LLMs to recover from them.  
这是我早期尝试修改 JSON 的实例的体现。我用的是 JQ 命令而不是 Python 脚本。JQ 用 0 或 1 的退出代码结束了所有可能的失败。这些输出是为终端设计的，而非让大型语言模型从终端中恢复。

![A drawing of an LLM represented as the brain, having access to different tools (represented as a saw, hammer and screwdriver) and those tools having access to the code. A big red cross in a discontinued line hinting the llm can't access directly the code](https://substackcdn.com/image/fetch/$s_!FMLc!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F29efd9f8-533f-4546-9969-c87ff4d8c1fb_1476x881.png)

A drawing of an LLM represented as the brain, having access to different tools (represented as a saw, hammer and screwdriver) and those tools having access to the code. A big red cross in a discontinued line hinting the llm can't access directly the code

One more thing worth noting: A “boring” codebase is better for agents. Stable APIs, predictable patterns, and simple architectures are far easier for agents to model than clever abstractions. Every layer of complexity you add to your codebase is a layer the agent has to navigate.  
还有一点值得一提：“无聊”的代码库对代理来说更好。稳定的 API、可预测的模式和简单的架构对智能体来说比巧妙的抽象更容易建模。你给代码库增加的每一层复杂性，都是代理必须穿越的一层。

Keep it simple.保持简单。

### 4\. Entropy Management (Garbage Collection)4. 熵管理（垃圾回收）

This is something most people skip. AI agents replicate patterns, including bad ones. Over time, your codebase accumulates “AI slop”: redundant logic, verbose implementations, subtly hallucinated variables that the model keeps copying because they exist in the context.  
这是大多数人跳过的内容。人工智能代理会复制模式，包括不良模式。随着时间推移，你的代码库积累了“AI 杂质”：冗余的逻辑、冗长的实现、模型不断复制的微妙幻觉变量，因为它们存在于上下文中。

Left unchecked, this entropy degrades the entire codebase. People call it context poisoning.  
如果不加以控制，这种熵会使整个代码库退化。人们称之为情境中毒。

Some people use this as an argument that AI is bad. But whenever I face a bad AI output, instead of judging if AI is good or bad for this task, I ask myself how can we make AI work here? The answer is usually adding another harness.  
有些人用这作为人工智能不好的论点。但每当我遇到糟糕的 AI 输出时，我不再评判 AI 在这项任务中是好是坏，而是问自己：我们如何让 AI 在这里发挥作用？答案通常是再加一个背带。

We can have a recurring cleanup agent. Think of it as garbage collection for your repo. For any implementation task, have a separate agent that scans the codebase, looks for drift from your golden principles, and fixes things before raising the PR. You can also execute this kind of agent on a schedule. Because you already designed other harnesses, like having unit tests and linters, you can allow AI to refactor code with confidence.  
我们可以安排一个常驻的清理人员。可以把它当作仓库的垃圾回收。对于任何实现任务，都应该有一个独立的代理扫描代码库，寻找偏离黄金原则的迹象，并在提高 PR 前修复问题。你也可以按计划执行这种代理。因为你已经设计了其他线束，比如单元测试和 linter，你可以让 AI 自信地重构代码。

It is the same concept as a “doc-gardening” agent that scans for stale documentation and updates it. Technical debt is called like this becuase it works like money debt. If you pay it daily, you stay solvent. If you let it accumulate, you end up spending a lot of time later.  
这和“文档园艺”代理扫描陈旧文档并更新的概念是一样的。技术债务之所以这样称呼，是因为它的工作原理类似于货币债务。如果你每天支付，你就能保持偿付能力。如果你让它积累，之后你会花很多时间。

The harness should include entropy management from day one, not as an afterthought.  
这个工具从一开始就应该包含熵管理，而不是事后才想到的。

To know where to apply the harnesses, I covered a 3-level framework for AI-assisted coding in this previous post:  
为了知道如何应用这些工具，我在之前的文章中介绍了一个 AI 辅助编码的三层框架：

---

## The Mindset Shift: From Prompts to Harness Engineering思维转变：从提示到束缚设计

The biggest change harness engineering requires is not technical. It is mental.  
线束工程最大的改变不是技术层面。这很精神化。

You stop writing prompts. You start designing environments. Your job is neither to write code nor to write the detailed prompts. It is to make the codebase **legible to the agent**. Every file name, every directory structure, every naming convention, every piece of documentation exists not just for human developers but for the autonomous agents that will read, modify, and extend the codebase at machine speed.  
你停止写提示。你开始设计环境。你的工作既不是写代码，也不是写详细的提示。目的是让代码库 **对代理来说是可读** 的。每一个文件名、每一个目录结构、每一个命名规范、每一份文档，不仅存在于人类开发者，也存在于能够以机器速度读取、修改和扩展代码库的自主代理。

Constraints stop being restrictions and start being **multipliers**. A custom linter you write once applies to every line of code the agent writes, deterministically, and forever. A structural test you build today catches every future violation automatically. You invest once, and the return compounds with every agent run. That is the leverage engineers had for humans, and we need it for AI agents.  
约束不再是限制，而是乘 **数** 。你写一次的自定义线条会被执行体写的每一行代码都确定性地应用，并且是永久的。你今天构建的结构测试会自动发现未来的所有违规。你投资一次，每次特工运行回报都会累计增加。这就是工程师对人类的影响力，而我们需要它来对付人工智能代理。

The engineers shipping the most code right now all converged on this independently. OpenAI’s internal team shipped one million lines of code and 1,500 PRs in five months using Codex with this approach. Anthropic’s Claude Code team has released 52 features in 50 days. My team at Amazon ships 100+ PRs per month. The patterns are the same: narrow the problem, use deterministic scripts at the execution boundary, enforce constraints mechanically, and make the codebase legible to the agent.  
目前发布最多代码的工程师们都独立地集中在这个问题上。OpenAI 内部团队在五个月内使用 Codex 发布了 100 万行代码和 1500 个 PR。Anthropic 的 Claude Code 团队在 50 天内发布了 52 个功能。我亚马逊团队每月发送 100+个永久居民。模式相同：缩小问题范围，在执行边界使用确定性脚本，机械性地强制约束，并使代码库对代理可读。

![Claude Shipping Calendar, Claude Release Notes](https://substackcdn.com/image/fetch/$s_!MA29!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F76faddcd-a8b9-4f8f-b177-c0b073428bc2_1840x2126.png)

source 资料来源

---

Learn the Building Blocks of AI Agents  
学习人工智能代理的基本要素

If you want the full guide, let me know your email below, and I’ll send you the free **“AI Agents Building Blocks”** guide inside the newsletter’s welcome email  
如果你想要完整指南，请在下方留言你的邮箱，我会在 通讯的欢迎邮件中免费发送 **《人工智能代理构建模块》** 指南

![Ebook cover of "AI Agents Building Blocks"](https://substackcdn.com/image/fetch/$s_!p39U!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1f389e59-901f-4e99-b38c-56bf136001a8_1628x2624.png)

Ebook cover of "AI Agents Building Blocks"

---

## Recap: Harness Engineering Key Concepts回顾：安全带工程关键概念

**What is harness engineering in AI?  
什么是人工智能中的线束工程？**

Harness engineering is the discipline of designing the systems, constraints, execution environments, and feedback loops that wrap around AI agents to make them reliable in production. Unlike prompt engineering, which focuses on a single model interaction, harness engineering governs the entire agent lifecycle, from state management to automated validation.  
机束工程是设计环绕人工智能代理的系统、约束、执行环境和反馈回路的学科，使其在生产环境中可靠。与专注于单一模型交互的提示工程不同，机用工程师工程管理整个代理生命周期，从状态管理到自动验证。

**How is harness engineering different from prompt engineering?  
线束工程和提示工程有什么不同？**

Prompt engineering crafts the input to the model in a single interaction. Harness engineering designs the entire environment the agent operates in: tools, guardrails, documentation structure, and automated feedback loops. The goal is reliable behavior across thousands of runs, not just one.  
提示工程通过一次互动为模型打造输入。利用器工程设计了代理运行的整个环境：工具、护栏、文档结构和自动化反馈循环。目标是在数千次运行中保持可靠的行为，而不仅仅是一次。

**Why do AI agents fail on large structured files like JSON?  
为什么 AI 代理在像 JSON 这样的大型结构化文件上会失败？**

Large JSON files exceed or crowd out the model’s context window, causing the agent to lose track of the surrounding structure. It may correctly modify the target node but corrupt adjacent keys, producing a broken file. The fix is a deterministic script that handles the file surgery, with the agent only providing the intent.  
大型 JSON 文件超出或挤出模型的上下文窗口，导致代理失去对周围结构的跟踪。它可能正确修改目标节点，但会破坏相邻键，导致文件损坏。修复是一个确定性脚本，负责处理文件操作，代理只提供意图。

**How do you build a simple AI agent harness?  
如何构建一个简单的 AI 代理框架？**

Start by narrowing the problem to one operation. Write deterministic scripts for the execution step. Wire the agent to tool-call those scripts instead of generating the output directly. Add a validation step that the agent cannot bypass (embed it in scripts if needed!). This three-part loop, intent to deterministic execution to validation, is the minimal viable harness.  
首先将问题缩小到一个操作范围。为执行步骤写确定性脚本。给代理布线让工具调用这些脚本，而不是直接生成输出。添加一个代理无法绕过的验证步骤（如有需要，可以嵌入脚本中！）。这个三部分循环，意图从确定性执行到验证，是最小可行的机束。

**What is an** `AGENTS.md` **file and why does it matter?**  
**什么是** `AGENTS.md` **文件，为什么重要？**

`AGENTS.md` is a file in your repository that tells an AI agent the rules, conventions, and architectural constraints of your codebase. It acts as the agent’s static context, injected at startup, so it knows your team’s norms without you having to repeat them in every prompt. Keep it short (under 100 lines) and use it as a table of contents pointing to deeper documentation.  
`AGENTS.md` 是你仓库中的一个文件，告诉 AI 代理代码库的规则、惯例和架构约束。它作为代理的静态上下文，启动时注入，因此它知道团队的规范，而不必你在每个提示中重复。保持简短（100 行以内），并用作目录，指向更深入的文档。

---

## Conclusion: The Harness IS the Product结论：安全带就是产品

The model is the easy part. Everyone has access to the same foundation models. GPT, Claude, Gemini, they are all remarkably capable. The harness is the hard part. The harness is what makes Claude Code, Cursor, and Codex ship production code instead of impressive demos. The harness is what separates a demo that impresses your manager from a production system that ships real code every day without breaking things.  
模型部分比较简单。每个人都能使用相同的基础模型。GPT、克劳德、双子座，他们都非常有能力。安全带才是难点。正是这种背带让 Claude Code、Cursor 和 Codex 能生成生产代码，而不是令人印象深刻的演示。线束是区分一个让你经理印象深刻的演示和每天发布真实代码且不破坏内容的生产系统的关键。

Here is what I want you to take away from this article:  
以下是我想让你从这篇文章中得到的启示：

- **Narrow the problem before you build the agent.** A scoped agent that does one thing well beats a general-purpose agent that does everything poorly.  
	在 **构建代理之前，先缩小问题范围。** 一个有定位的特工能做一件事，比一个什么都做得很差的通用特工要好。
- **Use deterministic scripts at the execution boundary.** Let the agent provide intent. Let the script provide the guarantee.  
	在 **执行边界使用确定性脚本。** 让代理人提供意图。让脚本提供保证。
- **Enforce constraints mechanically, not verbally.** If a rule matters, make it a linter, a test, or a validation step. Do not put it in a prompt and hope for the best.  
	**强制约束是机械的，而不是口头的。** 如果规则重要，可以做成 linter、测试或验证步骤。不要把它放在提示里然后抱着侥幸的希望。
- **Make the codebase legible to the agent, not just to humans.** Progressive disclosure, structured documentation, and the repo as the single system of record.  
	**让代码库对代理来说清晰可读，而不仅仅是对人类。** 渐进式披露、结构化文档以及作为单一记录系统的仓库。

The engineers who figure this out first will have an enormous advantage. Not because they have better models, but because they have better harnesses.  
那些最先发现这一点的工程师将拥有巨大的优势。不是因为他们有更好的型号，而是因为他们有更好的安全带。

---

If you read until this point, you have to read this other article with the AI concepts every software engineer needs to know in 2026:  
如果你读到这里，你必须读一篇关于 2026 年每位软件工程师都需要了解的 AI 概念的另一篇文章：