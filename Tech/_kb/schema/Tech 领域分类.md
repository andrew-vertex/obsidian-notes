# Tech 领域分类

定义 Tech 领域使用的主要子域。

## 子域

- `AI/`: AI 应用开发学习体系、大模型基础、Prompt/Context Engineering、RAG、MCP、评测、模型网关、AI 系统设计
- `Agent/`: AI agent 方法论、控制面、状态管理、guardrails、capabilities、handoff、evaluation、delivery workflow
- `Algorithms/`: 算法、题解、复杂度分析
- `Architecture/`: 通用架构原则、系统设计、分布式设计、性能
- `Backend/`: 数据库、搜索引擎、服务端框架、微服务、接口治理
- `DevOps/`: CI/CD、Docker、Kubernetes、Linux
- `Programming/`: 语言特性、编码模式、工程实践

## 放置规则

- 如果内容核心是“某个工具怎么安装/配置/使用”，优先去 `Tools/`
- 如果内容核心是“AI 应用开发的学习体系、模型调用、RAG、MCP、Prompt/Context、评测或生产系统设计”，优先去 `Tech/AI/`
- Java 后端转 AI 应用开发路线及其逐篇阅读笔记统一维护在 `Tech/AI/Java 转 AI 应用开发/`；`index.md` 作为稳定导航入口，专题文件按阶段子目录归档。
- 如果内容核心是“AI agent 的方法论、工程结构、运行控制、交付模式”，优先去 `Tech/Agent/`
- 如果内容跨多个知识域，再考虑在 root `wiki/` 做跨域 hub，而不是替代域内正式笔记
