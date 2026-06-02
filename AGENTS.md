# 知识库 Agent 规范

这个仓库是个人 Obsidian 知识库。Agent 的目标是把资料整理成长期可读、可检索、可维护的 Markdown 笔记，而不是只完成一次性回答。

## 目录约定

- `Clippings/` 是临时收集入口，默认不当作正式知识页。
- `_kb/` 是 vault 级控制面，存放跨域 raw、schema、wiki hub 和日志。
- `Tech/_kb/`、`Tools/_kb/` 是领域控制面；`Tech/`、`Tools/` 下的普通主题目录是正式笔记层。
- `Work/`、`Resume/`、`Thinking/` 等目录按现有主题边界维护，不要随意搬迁。

## 命名规则

- 正式笔记、wiki 长文、schema 长文、来源记录优先使用中文文件名，并尽量与 H1 或 frontmatter `title` 一致。
- `index.md`、`log.md`、`README.md` 保留固定英文名，作为 agent 和脚本稳定发现的控制入口。
- 历史导出、外部源码、URL slug、带 UUID 的 raw 文件可以保留原始文件名。
- 重命名笔记时必须同步更新 Obsidian wikilink、Markdown 相对链接、索引页和日志中的路径。

## 笔记格式

- 长篇正式笔记优先包含 frontmatter、H1、简短摘要、`## 目录`、`## Key Takeaways`、正文主体和 `## 参考来源`。
- 中文笔记保持中文表达；英文专有名词按技术社区常用写法保留。
- 不确定的信息标注 `需确认`，不要把推测写成事实。

## Wiki 工作流

- 新资料先进入 inbox 或最近的 `_kb/raw/`，不要直接覆盖原始材料。
- 导入资料时先读最近的 `_kb/index.md`，判断是更新旧页还是创建新页。
- 优先更新已有主题页，避免为相近主题创建重复笔记。
- 完成正式 ingest 后，更新对应领域的 `_kb/index.md`，并追加 `_kb/log.md`。
- 修改结构、模板或命名规则时，同步更新 `_kb/schema/`，必要时同步相关 note/wiki skill。

## Skill 使用

- 单页整理、扩写、补目录、补表格和 Mermaid 图时使用 `note-curator`。
- 资料提升、wiki 编排、索引维护、日志维护和 schema 调整时使用 `llm-wiki-curator`。
- 这两个 skill 的默认命名规则应与本仓库保持一致：正式内容用中文文件名，控制入口保留 `index.md`、`log.md`、`README.md`。
