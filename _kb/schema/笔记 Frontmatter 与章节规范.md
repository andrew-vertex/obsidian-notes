---
title: 笔记 Frontmatter 与章节规范
created: 2026-05-07
description: 当前 Notes vault 的正式笔记 frontmatter 与章节结构约定。
tags:
  - schema
  - obsidian
  - frontmatter
  - note-style
---
# 笔记 Frontmatter 与章节规范

这份规则用于约束 `note-curator` 和后续人工整理输出的正式笔记格式。

## 目录

- [Key Takeaways](#key-takeaways)
- [推荐 frontmatter](#推荐-frontmatter)
- [章节结构](#章节结构)
- [命名与标签](#命名与标签)
- [适用范围](#适用范围)
- [参考来源](#参考来源)

## Key Takeaways

- 正式知识型笔记建议统一使用 frontmatter。
- 推荐属性是 `title`、`created`、`description`、`tags`。
- 长篇技术参考笔记优先包含 `目录`、`Key Takeaways`、正文主体、`参考来源`。
- 索引页可以更轻，但依然建议保留 `tags` 或其他最小元数据。

## 推荐 frontmatter

```yaml
---
title: Exact note title
created: 2026-05-07
description: One-line summary
tags:
  - primary-topic
  - secondary-topic
---
```

规则：

- `title`: 与 H1 保持一致
- `created`: 使用 `YYYY-MM-DD`
- `description`: 一句话摘要，便于预览、搜索和理解
- `tags`: 使用稳定、可复用的主题词，不要滥加

## 章节结构

对于长篇技术参考笔记，默认结构为：

1. frontmatter
2. `# Title`
3. 1 到 3 句摘要
4. 可选的官方地址区块
5. `## 目录`
6. `## Key Takeaways`
7. 主题正文
8. `## 最佳实践` 或 `## 注意点与边界`
9. `## 参考来源`

如果是概念型、对比型或方法论型笔记，可在正文中使用：

- `## Overview`
- `## Key Concepts`
- `## Comparison Table`
- `## Architecture or Flow`
- `## Principles and Mechanisms`
- `## Practical Usage`

## 命名与标签

- 正式笔记文件名优先使用中文，并尽量与 H1 或 `title` 保持一致
- `index.md`、`log.md`、`README.md` 这类控制入口保留固定英文名，方便 agent 和脚本稳定发现
- 旧的英文 kebab-case 文件名只用于外部源码、代码路径、URL slug 或历史导出文件
- `tags` 优先反映主题，而不是情绪或临时任务状态
- 索引页建议加 `index` 标签

## 适用范围

- `Tools/` 下的安装、配置、工作流与最佳实践类笔记
- `Tech/` 下的概念、架构、后端、DevOps 与编程类知识页
- 由 `note-curator` 或 `llm-wiki-curator` 生成的正式知识页

## 参考来源

- Obsidian Properties: <https://help.obsidian.md/properties>
- Obsidian YAML frontmatter: <https://help.obsidian.md/advanced-topics/yaml-frontmatter>
