# Tech 领域日志

Tech 领域 ingest、query、lint 的追加式维护日志。

## [YYYY-MM-DD] seed | initialize domain knowledge base
- created _kb/raw and _kb/schema
- prepared domain index and domain log

## [2026-05-07] ingest | harness engineering article
- promoted the clipping into `Tech/_kb/raw/Harness Engineering 文章来源记录.md`
- created canonical note `Tech/Agent/AI Agent 的 Harness Engineering.md`
- updated `Tech/_kb/index.md` and `Tech/_kb/schema/Tech 领域分类.md`

## [2026-05-11] ingest | elasticsearch notion export
- promoted the original Notion export from `Temp/` into `Tech/_kb/raw/elasticsearch-notion-export/`
- added source record `Tech/_kb/raw/Elasticsearch Notion 导出来源记录.md`
- created the canonical Elasticsearch page set under `Tech/Backend/DateBase/elasticsearch/`
- updated `Tech/_kb/index.md` so the Tech domain can route directly to the Elasticsearch knowledge set

## [2026-06-03] maintenance | 中文文件名规范更新
- renamed Tech formal notes and source records to Chinese filenames
- updated `Tech/_kb/index.md`, `Tech/_kb/log.md`, and Tech schema links

## [2026-06-15] ingest | JavaGuide AI 应用开发知识体系
- added `Tech/AI/` as the AI 应用开发 formal note layer
- created source record `Tech/_kb/raw/JavaGuide AI 应用开发知识体系来源记录.md`
- created canonical note `Tech/AI/AI 应用开发学习体系.md`
- updated `Tech/_kb/index.md` and `Tech/_kb/schema/Tech 领域分类.md`

## [2026-06-21] ingest | Java 后端转 AI 应用开发学习路线
- preserved the original reading note `Inbox/Java-to-AI-roadMap - AI 应用开发与 Agent 学习路线.md`
- created source record `Tech/_kb/raw/JavaGuide Java Go 开发者 AI 应用开发路线来源记录.md`
- created the canonical workspace `Tech/AI/Java 转 AI 应用开发/` with a roadmap, progress checklist, and staged topic skeletons
- linked the previous `Tech/AI/AI 应用开发学习体系.md` overview to the new workspace
- updated `Tech/_kb/index.md` and `Tech/_kb/schema/Tech 领域分类.md`

## [2026-06-22] ingest | LLM 运行机制
- added source record `Tech/_kb/raw/JavaGuide LLM 运行机制来源记录.md`
- expanded the planned LLM fundamentals skeleton into `Tech/AI/Java 转 AI 应用开发/01-大模型基础/LLM 运行机制：Token、上下文窗口与采样参数.md`
- normalized time-sensitive model data into verification notes and added engineering decision tables, Mermaid flows, observability fields, and validation experiments
- updated the Java-to-AI workspace links after aligning the filename with the formal note title

## [2026-06-22] ingest | 大模型结构化输出
- added source record `Tech/_kb/raw/JavaGuide 大模型结构化输出来源记录.md`
- expanded and renamed the planned skeleton as `Tech/AI/Java 转 AI 应用开发/01-大模型基础/大模型结构化输出：从 JSON 契约到 Function Calling 落地.md`
- verified provider-specific boundaries against current OpenAI, Anthropic, Gemini, MCP, and JSON Schema documentation
- separated model-facing parameters from trusted runtime metadata and added Java validation, security, retry, observability, and rollout guidance
- updated all Java-to-AI workspace backlinks to the canonical filename

## [2026-06-22] ingest | Hello-Agents 第一章初识智能体
- moved the existing chapter note from `AI/AI Developer/Agent/` into the canonical Tech AI learning layer
- created `Tech/AI/Hello-Agents/index.md` as the 16-chapter course map and progress entry
- rewrote `第一章 初识智能体.md` with frontmatter, TOC, key takeaways, comparison tables, Mermaid diagrams, safer tool-call patterns, production boundaries, and evaluation guidance
- added source record `Tech/_kb/raw/Hello-Agents 第一章来源记录.md`
- linked the course and chapter from `Tech/_kb/index.md`

## [2026-06-22] ingest | 大模型提示词工程
- added source record `Tech/_kb/raw/JavaGuide 大模型提示词工程来源记录.md`
- expanded and renamed the planned Prompt skeleton as `Tech/AI/Java 转 AI 应用开发/02-Prompt 与上下文/大模型提示词工程（Prompt Engineering）是什么？提示词技巧有哪些？.md`
- restructured the topic around prompt anatomy, six technique families, evidence handling, evaluation, lifecycle governance, injection defense, and Agent context boundaries
- verified the Spring AI example against the current 2.0.0 reference and normalized reasoning-model guidance against current provider documentation
- updated all Java-to-AI workspace backlinks to the canonical filename
