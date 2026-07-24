---
title: "AI 应用学习路线"
created: 2026-07-24
updated: 2026-07-24
tags:
  - ai
  - agent
  - llm
type: guide
status: distilled
---

阶段一：LLM基础认知：
 - Transformer 架构 qkv 和 Attention 机制
 - Token - 成本
 - 上下文窗口 
 - Temperature 和 Top-p
 - 幻觉
	推荐网站：[大语言模型智能体简介 | Prompt Engineering Guide]([https://www.promptingguide.ai/zh/research/llm-agents](https://gw-c.nowcoder.com/api/sparta/jump/link?link=https%3A%2F%2Fwww.promptingguide.ai%2Fzh%2Fresearch%2Fllm-agents))
阶段二：AI 应用基础
- Prompt Engineering
	- 结构化输出 JSON
	- Role + Task + COnstraint
- Rag 和 向量数据库 - 商业化最广泛的技术
	- 私有数据
	- 幻觉
	- 实时更新
	奠基论文：[https://arxiv.org/abs/2005.11401](https://gw-c.nowcoder.com/api/sparta/jump/link?link=https%3A%2F%2Farxiv.org%2Fabs%2F2005.11401)  前沿方案：[https://arxiv.org/abs/2312.10997](https://gw-c.nowcoder.com/api/sparta/jump/link?link=https%3A%2F%2Farxiv.org%2Fabs%2F2312.10997) 帝王分析：[https://app.notion.com/p/RAG-354e766aea9f8049ad24c84cce32924a?pvs=21](https://gw-c.nowcoder.com/api/sparta/jump/link?link=https%3A%2F%2Fapp.notion.com%2Fp%2FRAG-354e766aea9f8049ad24c84cce32924a%3Fpvs%3D21)
 - 任务：知识库问答 Agent 
	- 上传文档  
	- 选取文档向量化（Embedding）  
	- LLM回答基于文档回答  
	- 这里对于这个llm的回答提示词，可以锻炼一下自己写，尝试写三个不同风格的版本，看看输出结果是否相同，然后还可以根据结果进行评测。 
- MCP
- skill
- Agent 的 Harness
	