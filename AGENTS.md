# 知识库 Agent 规范

这个仓库是个人 Obsidian 知识库，由 claude-obsidian 插件驱动。Agent 的目标是把资料整理成长期可读、可检索、可维护的 Markdown 笔记。

## 目录约定

- `Clippings/` 是临时收集入口（inbox），默认不当作正式知识页。
- `wiki/` 是 AI 驱动的知识库层 — 所有正式知识页、MOC、概念、实体、来源摘要的存放处。
- `.raw/` 是不可变来源文件层（articles/transcripts/screenshots/data/assets）。Claude 只读不写。
- `_templates/` 是 7 个笔记模板，定义每种笔记类型的结构契约。
- `Tech/`、`Tools/`、`Work/`、`Thinking/`、`Life/`、`Projects/` 等为旧笔记目录，逐步通过 ingest 迁移到 `wiki/`，最终降级为归档或删除。新内容直接进入 `wiki/`。

## 命名规则

- 正式笔记优先使用中文文件名，与 H1 或 frontmatter `title` 一致。
- `index.md`、`log.md`、`README.md`、`hot.md` 保留固定英文名，作为 agent 稳定发现的控制入口。
- 历史导出、外部源码、URL slug、带 UUID 的 raw 文件保留原始文件名。
- 重命名笔记时同步更新 Obsidian wikilink 和相关 MOC 索引。

## 笔记格式

- 正式笔记遵循 `_templates/` 下的 7 个模板结构（concept/howto/course/design/entity/source/question）。模板是参考框架而非模具——段落按实际内容动态调整。
- 所有笔记包含 frontmatter：`title`、`created`、`description`、`tags`、`layer`。
- 长篇笔记包含 `## 目录`、`## Key Takeaways`、正文主体、`## 参考来源`。
- 不确定的信息标注 `需确认`，不把推测写成事实。

## 设计原则（四合一）

1. **渐进式摘要** — 笔记按 Layer 1-3 递进加工。Layer 1 = 加粗关键句，Layer 2 = 高亮最佳部分，Layer 3 = 执行摘要。大多数笔记停留 Layer 1，只有被反复引用的才升级。
2. **模板即结构契约** — 同类型笔记 section 序列固定，AI 和人类都知道在哪读、在哪写。元素按需裁减。
3. **MOC 驱动导航** — 领域使用主动策划的内容地图（🎯进行中 / 📚已归档 / 🔮待探索），而非被动列表。
4. **可扫读** — 执行摘要在前、细节在后。表格和 emoji 做视觉锚点。

## 两层导航

- **MOC 层**（人类导航）：`wiki/overview.md`（Home MOC）→ 领域 MOC → 原子页
- **index 层**（AI/搜索）：`wiki/index.md` 保留全 vault 扁平索引

## Wiki 工作流（claude-obsidian 驱动）

### 摄入来源

1. 文件放入 `.raw/`，在 Claude 中说 `ingest [文件名]`。
2. Claude 读取来源 → 提取实体/概念 → 创建或更新 `wiki/` 下的页面 → 更新 MOC 索引 → 追加 `wiki/log.md` → 刷新 `wiki/hot.md`。
3. 优先更新已有页面，避免为相近主题创建重复笔记。

### 查询知识

1. Claude 先读 `wiki/hot.md`（热缓存）→ 再读 `wiki/index.md`（扁平索引）→ 最后钻入 3-5 个最相关页面。
2. 综合回答时使用 `[[wikilink]]` 引用来源页。

### 保存对话

- `/save` — 将当前对话保存为 wiki 笔记。

### 健康检查

- `lint the wiki` — 定期检查孤立页、死链、缺失来源段、过期声明。

### 自主研究

- `/autoresearch [主题]` — web 搜索 → 综合 → 归档进 wiki。

## Skill 使用

- 单页整理、扩写、补目录、补表格和 Mermaid 图：`note-curator`。
- 所有 wiki 生命周期操作（ingest/query/lint/save/index/log）：`claude-obsidian` 插件（`/wiki`, `ingest`, `/save`, `lint the wiki`, `/autoresearch`）。
- 深度思考：`/think [问题]`。
