# Canvas Output Guide

`visual` 模式首版只输出 Obsidian `.canvas` 文件。

## Diagram Types

| 类型 | 何时使用 | 核心节点 |
|------|---------|---------|
| `concept-map` | 概念、术语、关系最重要 | 概念节点、关系边、例子节点 |
| `timeline-map` | 时间顺序或论证顺序明显 | 阶段节点、事件节点、因果边 |
| `topic-map` | 系列导览、合集导航 | 主题组、视频节点、入口节点 |
| `argument-map` | 观点与论据结构明显 | 主张节点、证据节点、限制条件节点 |

优先规则：

- 单视频且概念密集：`concept-map`
- 单视频且论证链清晰：`argument-map`
- 系列 / 合集导航：`topic-map`
- 有时间演进：`timeline-map`

## Node Strategy

优先使用：

- `text`：知识点、术语、结论、摘要
- `group`：主题分区、阶段分区、系列模块
- `file`：指向同目录下的 Markdown 笔记或索引页（如果本次同时生成）

避免：

- 大段文字塞进单节点
- 用颜色代替结构
- 用外部图片做主要信息承载

## Minimum Canvas Shape

每个 `.canvas` 至少包含：

- 1 个标题节点
- 2 个以上内容节点
- 至少 1 条边

推荐结构：

1. 左上角标题节点
2. 中央主结构
3. 周边补充节点
4. 如果有文档输出，在右侧或底部放 `file` 节点链接到对应 Markdown

## Layout Rules

- 从左到右或从上到下保持单一主流向
- 同一层级节点大小尽量一致
- 主题组之间留足空白，避免重叠
- 使用 `group` 节点给系列/主题分区

## Naming

- 单视频：`[视频标题] - 概念图.canvas`
- 系列：`[系列名] - 主题图.canvas`
- 时间线：`[标题] - 时间线.canvas`
- 论证图：`[标题] - 论证图.canvas`

## Linkage

如果同批次生成了 Markdown 文档：

- `topic-map` 应至少链接 `index`
- `concept-map` 或 `argument-map` 可链接对应 `notes`
- `timeline-map` 可链接 `timeline` 文档

## Validation

生成 `.canvas` 时检查：

- JSON 顶层为 `nodes` / `edges`
- 每个节点有唯一 `id`
- 每条边的 `fromNode` / `toNode` 都存在
- 没有空文本节点
- 文件节点路径与输出目录一致
