# bilibili-transcript-organizer

[English](./README.md)

把 B 站视频、合集、分 P、收藏夹或字幕文件整理成结构化知识产物。

这个仓库提供一个 Claude Code skill，定位是**知识整理**，不是内容改写。它会保留来源引用，支持多视频综合整理，并可输出 Markdown 笔记和 Obsidian `.canvas` 画布。

<p align="center">
  <img src="./assets/preview-notes.svg" alt="结构化笔记预览" width="48%" />
  <img src="./assets/preview-canvas.svg" alt="Canvas 画布预览" width="48%" />
</p>

## 预览

示例输出见 [`examples/sample-notes.md`](./examples/sample-notes.md) 和 [`examples/sample-topic-map.canvas`](./examples/sample-topic-map.canvas)。

`notes` 预览：

```md
## 核心结论

- 系列把问题解释为结构性问题，而不是孤立事件。
- 能源、外交和国内政治在多期内容中相互作用。
- 后续视频不是简单重复，而是在修正和补全前面的判断。

## 主题脉络

### 历史背景
解释理解当前事件所需的长期背景。

### 跨视频演变
追踪哪些判断保持稳定，哪些在后续视频中发生变化。
```

`.canvas` 预览：

```json
{
  "nodes": [
    { "id": "topic", "type": "text", "text": "核心主题" },
    { "id": "factor-a", "type": "text", "text": "历史背景" },
    { "id": "factor-b", "type": "text", "text": "政策影响" }
  ],
  "edges": [
    { "id": "e1", "fromNode": "topic", "toNode": "factor-a" },
    { "id": "e2", "fromNode": "topic", "toNode": "factor-b" }
  ]
}
```

## 实际工作流

这是一个典型多视频系列的端到端流程：

### 输入

```text
整理这个 B站合集，先做完整 notes，如果合适再推荐 glossary 或 visual：
https://space.bilibili.com/<user-id>/lists/<season-id>?type=season
```

### Skill 会做什么

1. 使用 `bili` CLI 获取视频元数据和字幕
2. 为整个系列建立一份共享的知识结构
3. 先生成基础 `notes`
4. 当内容适合时，再推荐 `glossary`、`timeline`、`visual` 等附加输出

### 常见输出结果

- 一份系列级综合 `notes`
- 保留 BV 链接、分 P 标记或时间戳等来源引用
- 可选的 Obsidian `.canvas` 主题图
- 当概念较密集时，可选 `glossary`

### 示例结果形态

- [`examples/sample-notes.md`](./examples/sample-notes.md)
- [`examples/sample-topic-map.canvas`](./examples/sample-topic-map.canvas)

## 为什么做这个

B 站有很多高价值的长视频分析、课程和议题型系列，但原始观看体验更适合消费，不太适合：

- 长期记笔记
- 跨视频综合整理
- 提取概念和术语
- 建立时间线
- 沉淀成可复用知识库

这个 skill 就是为了解决这个缺口。

## 特性

- 永远先生成基础 `notes`
- 支持附加模式：`brief`、`glossary`、`timeline`、`compare`、`index`、`qa`、`cards`、`visual`
- 使用 `bili` CLI 获取 B 站资源
- 支持本地字幕文件作为降级输入
- 支持单视频、分 P、合集、收藏夹和系列
- 多视频可整理为一篇综合笔记或多篇主题笔记
- 支持 Obsidian `.canvas` 画布输出
- 明确避免改写、模仿风格和洗稿式输出

## 适用场景

- 分析类视频
- 历史 / 政治 / 经济 / 科技讲解
- 议题型系列
- 课程型内容
- Obsidian 或通用 Markdown 知识管理工作流

## 不适用场景

- 娱乐向短视频
- 单纯下载字幕
- 营销改写
- 模仿原作者文风

## 快速开始

### 1. 首选安装方式：`skills.sh`

推荐普通用户优先使用 `skills.sh` 安装：

```bash
npx skills add Mocooa/bilibili-transcript-organizer
```

也可以直接使用完整 GitHub URL：

```bash
npx skills add https://github.com/Mocooa/bilibili-transcript-organizer
```

### 2. 本地开发或手动安装

如果你想本地查看或修改这个 skill：

```bash
git clone https://github.com/Mocooa/bilibili-transcript-organizer.git
cd bilibili-transcript-organizer
claude skill install .
```

如果你的环境不支持这个命令，可以手动把仓库目录复制到本地 skill 目录。

### 3. 安装 `bili` CLI 以直接获取 B 站内容

先检查：

```bash
command -v bili
```

如果未安装，可使用以下任一方式：

```bash
uv tool install bilibili-cli
# or
pipx install bilibili-cli
# or
python3 -m pip install --user bilibili-cli
```

验证：

```bash
bili --help
```

如果你不想安装 CLI，也可以直接输入本地 `.srt`、`.txt` 或 `.md` 字幕文件。

## 输出模式

`notes` 永远是基础产物，其他模式是在其上的附加视图。

| 模式 | 作用 |
|------|------|
| `notes` | 基础结构化笔记 |
| `brief` | 一页摘要 |
| `glossary` | 术语表 |
| `timeline` | 时间线或论证顺序 |
| `compare` | 多视频 / 多观点对比 |
| `index` | 系列索引页 |
| `qa` | 问答索引 |
| `cards` | 原子知识卡片 |
| `visual` | Obsidian `.canvas` 画布 |

`visual` 的 v1 只支持 Obsidian `.canvas`，不支持 Excalidraw。

## 示例提示词

```text
整理这个 B站视频：BV1ABcsztEcY
```

```text
把这 3 个 B站视频按 compare 模式整理，重点看观点分歧
```

```text
把这个合集整理成 glossary，输出关键术语和首次出现位置
```

```text
把这个系列整理成 visual 模式，做一个 canvas 主题图
```

```text
我有一些导出的字幕文件，按 cards 模式整理成知识卡片
```

## 多视频处理规则

对于多视频输入，这个 skill 不会机械地按数量拆分。

它会先判断主题复杂度，再决定输出为：

- 一篇综合 `notes`
- 多篇主题 `notes`

如果两种方式都合理，它会主动询问用户偏好。

文档数量和篇幅没有硬性上限，输出结构应服从内容结构和用户偏好。

## 仓库结构

```text
bilibili-transcript-organizer/
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── LICENSE
├── assets/
│   ├── preview-canvas.svg
│   └── preview-notes.svg
├── examples/
│   ├── sample-notes.md
│   └── sample-topic-map.canvas
└── references/
    ├── modes.md
    ├── templates.md
    └── canvas.md
```

## 文件说明

- `SKILL.md`：主 skill 指令
- `README.md`：英文 README
- `assets/preview-notes.svg`：README 中的结构化笔记预览图
- `assets/preview-canvas.svg`：README 中的 `.canvas` 预览图
- `examples/sample-notes.md`：公开示例笔记
- `examples/sample-topic-map.canvas`：公开示例画布
- `references/modes.md`：模式语义、降级规则和交付方式
- `references/templates.md`：Markdown 输出模板
- `references/canvas.md`：`.canvas` 输出规则

## FAQ

**推荐的安装方式是什么？**  
优先用 `skills.sh`：`npx skills add Mocooa/bilibili-transcript-organizer`。Clone / 手动安装更适合本地开发或自定义。

**必须使用 Obsidian 吗？**  
不需要。Markdown 输出在任何地方都能用，`.canvas` 主要面向 Obsidian Canvas。

**如果出现 `bili: command not found` 怎么办？**  
安装 `bilibili-cli`，或者改用本地字幕文件输入。

**它会把视频内容改写成“原创文章”吗？**  
不会。这个 skill 的定位是知识整理和索引，不是改写。

**如果单视频请求了 `compare` 会怎样？**  
它仍然会先生成 `notes`，然后平滑降级，并推荐更合适的 `timeline`、`glossary` 或 `visual`。

**它怎么决定一篇笔记还是多篇笔记？**  
根据主题复杂度，以及必要时用户明确表达的偏好。

## License

MIT
