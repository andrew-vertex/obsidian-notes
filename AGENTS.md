# 知识库 Agent 规范

这个仓库是个人 Obsidian 知识库。任何进入此目录的 AI agent 的目标都是一致的：把资料整理成长期可读、可检索、可维护的 Markdown 笔记，而不是只完成一次性回答。

以下规范不依赖任何特定插件或工具——所有操作都可以用基本的文件读写和搜索能力完成。

## 目录约定

- `Clippings/` — 临时收集入口（inbox），不当作正式知识页。
- `wiki/` — 知识库的导航与知识建模层：存放 MOC（内容地图）、索引、概念卡、实体卡、来源摘要、日志和热缓存；它通过 wikilink 连接各主题目录中的笔记，而不是所有正式笔记的默认物理位置。
- `.raw/` — 不可变来源文件层（articles / transcripts / screenshots / data / assets）。只读，不修改原始文件。
- `_templates/` — 7 个笔记模板（concept / howto / course / design / entity / source / question），定义每种笔记类型的结构契约。
- `Tech/`、`Tools/`、`Work/`、`Thinking/`、`Life/`、`Projects/`、`AI/`、`Resume/` — 内容的主分类目录。新笔记应优先按主题归入这些既有目录及其子目录（例如 macOS 工具指南放入 `Tools/Mac/`）；`wiki/` 中保留指向它们的 wikilink，使笔记能从知识地图被发现，无需为了导航而搬迁文件。

## 命名规则

- 正式笔记优先使用中文文件名，与 H1 或 frontmatter `title` 一致。
- `index.md`、`log.md`、`README.md`、`hot.md` 保留固定英文名，作为稳定发现的控制入口。
- 历史导出、外部源码、URL slug、带 UUID 的文件保留原始文件名。
- 重命名笔记时同步更新所有指向它的 `[[wikilink]]` 和相关 MOC 索引。

## 笔记格式

- 正式笔记遵循 `_templates/` 下的 7 个模板结构。模板是参考框架而非模具——段落按实际内容动态调整。创建新笔记时：先读对应模板了解结构骨架，再填充内容。
- 所有笔记包含 frontmatter：`title`、`created`、`description`、`tags`、`layer`。
- 长篇笔记包含 `## 目录`、`## Key Takeaways`、正文主体、`## 参考来源`。
- 不确定的信息标注 `需确认`，不把推测写成事实。

## 设计原则（四合一）

1. **渐进式摘要** — 笔记按 Layer 1-3 递进加工。Layer 1 = 加粗关键句，Layer 2 = 高亮最佳部分，Layer 3 = 用自己话写执行摘要。大多数笔记停留 Layer 1。`layer` 字段写入 frontmatter 表示当前加工深度。
2. **模板即结构契约** — 同类型笔记 section 序列固定，agent 和人类都知道在哪读、在哪写。元素按实际内容裁减。
3. **MOC 驱动导航** — 领域使用主动策划的内容地图（🎯 进行中 / 📚 已归档 / 🔮 待探索），而非被动列表。
4. **可扫读** — 执行摘要在前、细节在后。表格和 emoji 做视觉锚点。

## 两层导航结构

- **MOC 层**（人类导航）：从 `wiki/overview.md`（Home MOC）进入 → 领域 MOC（`wiki/learning/_index.md`、`wiki/work/_index.md`、`wiki/thinking/_index.md`、`wiki/goals/_index.md`）→ 具体知识页。
- **index 层**（搜索/快速定位）：`wiki/index.md` 保留全 vault 扁平索引。

## 核心工作流

### 摄入来源（Ingest）

当用户提供新的文章、视频笔记、截图、链接等资料时：

1. 将原始文件放入 `.raw/` 对应子目录（articles / transcripts / screenshots / data）。
2. 完整读取来源内容。
3. 提取其中的实体（人物、组织、工具、产品）和概念（想法、模式、框架、技术原理）。
4. 在 `wiki/sources/` 创建来源摘要页。参考 `_templates/source.md` 的结构。
5. 为每个重要实体在 `wiki/entities/` 创建或更新页面。参考 `_templates/entity.md`。
6. 为每个重要概念在 `wiki/concepts/` 创建或更新页面。参考 `_templates/concept.md`。
7. **优先更新已有页面**，避免为相近主题创建重复笔记。如果已有页面覆盖了相同概念，合并而非新建。
8. 将主笔记归入最匹配的主题目录；如其具有导航价值，再更新相关领域 MOC（`wiki/learning/_index.md` 等），用 wikilink 加入对应分区（🎯/📚/🔮）。
9. 在 `wiki/log.md` 顶部追加操作记录：日期、操作类型、影响的页面列表、关键收获一句话。
10. 重写 `wiki/hot.md`：更新当前焦点、最近变化、待处理事项。保持 ~500 字以内。

### 查询知识（Query）

当用户提出问题时：

1. 先读 `wiki/hot.md` — 热缓存可能已经有答案。
2. 再读 `wiki/index.md` — 找到相关页面的文件名。
3. 打开 3-5 个最相关页面（不要超过 10 个，控制上下文消耗）。
4. 综合回答，使用 `[[page-name]]` 或 `[[path/page-name|显示名]]` 引用相关笔记页面。
5. 如果回答有长期价值，主动提出将其保存为 `wiki/questions/` 下的问答页。参考 `_templates/question.md`。
6. 如果发现知识缺口（某个主题没有对应页面），主动告知用户。

### 保存对话（Save）

当用户要求保存当前讨论时：

1. 分析对话中具有长期价值的内容（决策、概念澄清、方法论、对比结论）。
2. 按内容类型选择最合适的模板（concept / howto / design / question）。
3. 先根据主题选择既有内容目录；仅 MOC、索引、概念/实体/来源卡、日志和热缓存进入 `wiki/`。
4. 如有长期导航价值，更新相关领域 MOC；随后更新 `wiki/log.md`。

### 健康检查（Lint）

定期或按需执行：

1. 检查 `wiki/` 下所有 `[[wikilink]]` 指向的页面是否存在，列出死链。
2. 检查是否有孤立页面（没有任何其他页面链接到它）。
3. 检查页面是否缺少 `## 参考来源` 或 `## Sources` 段。
4. 检查是否有标注 `layer: 1` 超过 30 天未更新的页面（可能是被遗忘的半成品）。
5. 检查是否有互相矛盾的声明（如两个页面对同一概念给出不同定义），标注 `> [!contradiction]`。
6. 将检查结果写入 `wiki/meta/lint-report-YYYY-MM-DD.md`。

### 自主研究（Research）

当用户要求深入研究某个主题时：

1. 搜索 web 获取相关资料。
2. 抓取高质量来源内容。
3. 按 Ingest 流程将研究主笔记归档到匹配的主题目录，并在 `wiki/` 创建必要的来源、概念或导航链接。
4. 创建研究综合页，汇总关键发现、矛盾和开放问题。

## 跨会话上下文

- 每次进入 vault 时，先读 `wiki/hot.md` 恢复最近上下文（~500 字）。
- 每次离开 vault 前，更新 `wiki/hot.md` 反映最新状态：当前焦点、最近变化、待处理事项、活跃线索。
- hot.md 是缓存，不是日志——完全重写而非追加。

## 推荐工具

以下为可选增强，不影响基本工作流：

- **note-curator** skill — 可复用的单页整理/格式化 skill。用于将散乱笔记整理为标准结构、补充对比表格和 Mermaid 图。
- **claude-obsidian** 插件 — 如当前 agent 环境支持，可使用其内置的 `/wiki`、`ingest`、`/save`、`lint the wiki`、`/autoresearch` 命令快捷执行上述工作流。不支持时按本文档手动执行即可。

## 快速上手

第一次进入此 vault 的 agent，按以下顺序了解全局：

1. 读 `wiki/overview.md` — 了解 vault 用途和当前焦点
2. 读 `wiki/hot.md` — 获取最近上下文
3. 读 `wiki/index.md` — 了解有哪些页面
4. 浏览 `_templates/` — 了解笔记结构约定
5. 回到本文档 — 需要执行具体操作时查阅对应工作流
