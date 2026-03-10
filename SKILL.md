---
name: bilibili-transcript-organizer
description: >
  整理 B站视频为可检索的知识材料。Use when the user wants to turn Bilibili videos, BV IDs, bilibili.com links,
  space.bilibili.com pages, multi-part videos, collections, favorites, or subtitle files into structured knowledge outputs.
  Trigger for organization intent such as 整理, 归纳, 学习笔记, 索引, 术语表, 时间线, 对比, 问答索引, cards, canvas, 画布, 概念图, 主题图, knowledge map.
  Supports output modes: notes, brief, glossary, timeline, compare, index, qa, cards, and visual Canvas maps.
  Does NOT trigger for merely watching a video, downloading subtitles only, non-Bilibili content, rewriting in the creator's style, marketing copy, or laundering-style rewrites.
---

# Bilibili 知识整理

把 B站视频、合集、分P、收藏夹或已有字幕整理为可沉淀、可检索、可引用的知识材料，而不是再创作内容。

## 定位与边界

- **目标**：学习、研究、索引、知识管理
- **允许输出**：笔记、摘要、术语表、时间线、对比表、索引页、问答索引、知识卡片、`.canvas` 主题图
- **禁止输出**：模仿原作者风格的改写稿、营销化二创、把原内容伪装成原创文章
- **来源要求**：所有模式都保留视频链接、BV/合集信息，以及期数或时间戳引用

## 输出模式

`notes` 不是可选默认值，而是**所有输入的基础产物**。单视频、多视频、分P、合集、系列都先产出 `notes`，其他模式是在 `notes` 之上追加的附加视图。

| 模式 | 适用场景 | 输出 |
|------|---------|------|
| `notes` | 所有任务的基础产物 | 结构化 Markdown 笔记 |
| `brief` | 想快速把握内容 | 一页摘要 |
| `glossary` | 概念密集、术语多 | 术语表 |
| `timeline` | 有明显时间线、演进顺序、论证顺序 | 时间线文档 |
| `compare` | 多视频、多 UP、多观点对照 | 对比文档 |
| `index` | 6+ 视频或明确系列导航 | 索引页 |
| `qa` | 希望检索问题和答案 | 问答索引 |
| `cards` | 学习和复习 | 原子知识卡片 |
| `visual` | 需要画布式知识图 | `.canvas` 文件 |

### 模式选择规则

1. 无论用户是否指定模式，都先交付 `notes`。
2. 用户显式说出 `compare`、`timeline`、`cards`、`canvas` 等模式词时，在 `notes` 之外追加该模式。
3. 用户没有指定其他模式时，交付 `notes`，并根据内容特点推荐 1-2 个适合的附加模式。
4. 如果输入是 6+ 视频、明确合集/系列、或需要导航结构，可在主输出之外自动补 `index`。
5. `visual` 首版只输出 Obsidian `.canvas`，不输出 Excalidraw。

### 不适合当前输入的模式

如果用户指定了和输入规模明显不匹配的模式，不要生硬失败，按以下方式降级：

- 单视频指定 `compare`：仍然先交付 `notes`，说明 `compare` 更适合多视频对照，并改为推荐 `timeline`、`glossary` 或 `visual`
- 非系列输入指定 `index`：仍然先交付 `notes`，说明当前输入不需要独立索引页
- 内容非常短却指定 `cards` / `qa`：可以减少产出数量，但保留 `notes`
- 任意情况下都不要因为附加模式不合适而跳过基础 `notes`

### 推荐附加模式

在交付 `notes` 后，按内容特征推荐额外模式：

- 概念密集、术语多：推荐 `glossary`
- 论证链或因果链清晰：推荐 `timeline` 或 `visual`
- 多视频存在明显观点差异：推荐 `compare`
- 系列 / 合集需要导航：推荐 `index`
- 适合复习、记忆：推荐 `cards`
- 适合检索常见问题：推荐 `qa`

## 输入形式

支持以下输入：

- 单个视频链接或 BV 号
- 合集 / 系列链接
- UP 主主页下的某个系列
- 分P视频
- 收藏夹
- 多个独立视频链接
- 本地 `.md` / `.txt` / `.srt` 字幕文件
- 用户在对话中粘贴的字幕文本

## 第一步：依赖与输入检查

当输入需要直接从 B站提取字幕时，先检查 `bili` 命令：

```bash
command -v bili >/dev/null 2>&1
```

处理规则：

1. **`bili` 已安装**：继续提取字幕和元数据。
2. **`bili` 未安装，但输入是本地字幕或粘贴文本**：跳过安装，直接处理。
3. **`bili` 未安装，且输入依赖在线提取**：不要继续盲跑 `bili ...`。明确说明缺少 CLI，并给出安装或降级路径：

```bash
uv tool install bilibili-cli
# 或
pipx install bilibili-cli
# 或
python3 -m pip install --user bilibili-cli
```

如果用户不想安装，改为让用户提供已导出的 `.srt` / `.txt` / `.md` 字幕文件。

## 第二步：收集并标准化内容

无论输入是什么，都先整理为共享的知识中间层，再渲染成具体产物。中间层至少包含：

- **源信息**：标题、BV/URL、UP 主、合集/分P关系、期数、发布时间
- **片段信息**：时间戳、片段文本、片段编号
- **知识信息**：核心论点、术语、案例/数据、关键实体、主题标签
- **引用信息**：每个结论对应的视频期数或时间戳
- **聚合信息**：跨视频关系、主题分组、覆盖状态、无字幕标记、`shared_topics`、`topic_evolution`、`agreements`、`disagreements`、`cross_refs`

### 提取建议

- 单视频：直接提取字幕并建立一个中间结构
- 多视频 / 合集：先拿到视频列表与元数据，再分批提取字幕
- 分P视频：每个分P都作为独立单元进入中间层，同时保留父视频关系
- 本地字幕文件：直接读取并估算规模，再写入中间层

如果是大合集，可并行分批提取；不支持并行的环境则串行处理。

## 第三步：分析与组织

从中间层做组织，而不是从原始字幕直接写最终文档：

- 判断内容规模：单视频、小批量、中等系列、大型合集
- 识别主题聚类、主线与支线
- 记录共识、分歧、因果关系、时间演进
- 标记无字幕内容与低质量片段，避免把缺失信息写成确定结论

### 多视频综合整理规则

多视频、分P、合集、系列默认做**综合整理**，而不是把每个视频机械地各写一篇孤立笔记。优先做以下工作：

- 识别跨视频的重复主题与共同主线
- 合并相近概念，避免同义内容反复出现
- 记录观点延续、修正和前后变化
- 提炼跨视频共识、分歧与证据强弱
- 追踪术语在不同视频中的首次出现与后续演变

只有当用户明确要求“逐个视频分别输出”时，才按视频拆开生成独立结果。

在开始写多视频结果之前，先判断主题复杂度，并主动确认用户想要的组织方式：

- **一篇综合 `notes`**：适合主题集中、主线明确、用户只想有一个入口文档
- **多篇主题 `notes`**：适合主题跨度大、支线明显、用户希望分主题沉淀

如果从内容看两种方式都合理，就直接询问用户要哪种；不要仅凭视频数量机械决定。

多视频输入时，优先保证：

- **覆盖完整**：所有有字幕视频都被纳入
- **引用清晰**：重要论点可回溯到期数或时间戳
- **内聚组织**：同主题内容放在一起，跨主题内容用引用连接

### 分流原则

- **单视频**：生成 1 篇 `notes`
- **多视频 / 分P / 合集 / 系列**：先看主题复杂度，再结合用户偏好决定是 1 篇综合 `notes` 还是多篇主题 `notes`
- **明显可比的多视角输入**：在 `notes` 之外追加 `compare`
- **需要导航结构时**：补 `index`

不要给文档数量和篇幅设置硬上限。输出几篇、每篇多长，应以内容结构和用户偏好为准，而不是固定规则。

## 第四步：按模式渲染

### Markdown 模式

`notes`、`brief`、`glossary`、`timeline`、`compare`、`index`、`qa`、`cards` 的具体结构见 [references/modes.md](references/modes.md) 和 [references/templates.md](references/templates.md)。

默认要求：

- 语言跟随视频内容原始语言
- 重要结论标注来源
- 优先结构化呈现：标题层级、表格、列表、索引
- 不输出“像原创文章一样的改写稿”
- `notes` 永远先生成；其他 Markdown 模式都视为附加视图

### Visual 模式

`visual` 模式首版输出 `.canvas` 文件。何时选用哪种图，见 [references/canvas.md](references/canvas.md)。

默认支持 4 种图：

- `concept-map`
- `timeline-map`
- `topic-map`
- `argument-map`

规则：

- 概念关系强：优先 `concept-map`
- 系列导览强：优先 `topic-map`
- 时间演进强：优先 `timeline-map`
- 观点-论据结构强：优先 `argument-map`

## 第五步：输出路径与命名

优先级：

1. 用户明确指定路径
2. 检测 Obsidian vault
3. 无 vault 时询问或使用用户指定目录

推荐路径：

- 单视频：`<vault>/Clippings/<视频标题>.md`
- 系列：`<vault>/Clippings/<UP主名>/`
- 画布：与同批次文档放在同一目录，如 `<系列名> - 主题图.canvas`

## 第六步：验证与收尾

在交付前至少检查：

- 有字幕的视频是否全部被覆盖
- 关键结论是否保留来源引用
- `index` 中的文档名和链接是否一致
- `visual` 模式生成的 `.canvas` 是否包含合法的 `nodes` / `edges`
- 没有把原视频内容改写成“原创口吻文章”

向用户报告：

- 创建了哪些文件
- 已先交付 `notes`
- 额外生成了哪些模式
- 推荐继续尝试哪些附加模式
- 哪些视频无字幕或被跳过
- 输出目录在哪里

### 统一交付话术

收尾时优先用这套顺序表达：

1. **已生成**：基础 `notes`
2. **额外生成**：本次追加的模式（如 `compare` / `index` / `visual`）
3. **推荐继续尝试**：最适合当前内容的 1-2 个附加模式
4. **补充说明**：无字幕、模式降级、跳过项、输出位置

## 何时读取 references

- 想知道模式差异与自动选择：读 [references/modes.md](references/modes.md)
- 要生成具体 Markdown 结构：读 [references/templates.md](references/templates.md)
- 要生成 `.canvas`：读 [references/canvas.md](references/canvas.md)
