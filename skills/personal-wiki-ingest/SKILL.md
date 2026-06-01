---
name: personal-wiki-ingest
description: 当用户要把一份或一批原始资料纳入个人 LLM Wiki 的正式知识流时使用这个 skill。适用于处理 raw/inbox 或 raw/library 中的来源、把资料整理进 library、登记 raw-index、创建或更新 wiki source page、同步必要 knowledge pages、重新 ingest 已有来源。不要用于初始化骨架、纯查询回答、lint 巡检、或不围绕具体 raw 来源的普通页面编辑。
---

# Personal Wiki Ingest

这个 skill 用于把 `raw/inbox/` 或 `raw/library/` 中的具体资料纳入正式知识流。

ingest 的核心不是“搬文件”，而是完成一条可追踪链路：

```text
raw/inbox/ 或 raw/library/
-> raw/library/
-> raw-index.md
-> wiki/sources/
-> 必要的 wiki/knowledge/ 或 wiki/syntheses/
-> index.md
-> log.md
```

ingest 必须主动建立知识链。不能只生成孤立的 source page，也不能把关联留给 query 才处理。

ingest 的关键动作是：把本次明确 ingest 的来源与 vault 中已有 `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/` 放在同一个分析集合中，整体抽取方法论、概念、问题、决策原则、操作经验和跨文档关联。

本 skill 以 `personal-wiki-init` 定义的最小 schema 为准：

- raw 只按需使用 `sources/`、`notes/`
- wiki 只按需使用 `sources/`、`knowledge/`、`syntheses/`
- `raw-index.md` 是唯一官方 raw ingest 状态表
- 页面结构来自 `system/workflow.md` 的 `Page Shapes`
- 不使用 `topics/`、`people/`、`concepts/`、`questions/` 作为目录
- wiki 页面之间的关系必须写成 Obsidian `[[...]]` 双链
- ingest 必须基于本次来源和已有 wiki 的整体语料主动抽取方法论、概念、问题与关联关系

## 目录职责

| 路径 | 职责 |
| --- | --- |
| `raw/inbox/sources/` | 待 ingest 的外部来源原文。 |
| `raw/library/sources/` | 已纳入正式知识流的外部来源原文。 |
| `raw/inbox/notes/` | 待处理的用户原始笔记、摘录、临时记录。 |
| `raw/library/notes/` | 已纳入正式知识流的用户原始笔记、摘录、临时记录。 |
| `wiki/sources/` | 来源追踪页。每页对应一个 raw item，记录“这个 raw item 说了什么”，包括 `raw/library/sources/` 和 `raw/library/notes/`。 |
| `wiki/knowledge/` | 结构化知识页。承载从来源中抽取出的可复用方法论、概念、问题、决策原则或操作经验。 |
| `wiki/syntheses/` | 综合页。承载跨多个 sources / knowledge pages 的比较、共识、分歧和阶段性结论。 |

关键边界：

- raw `notes/` 是用户原始输入，仍属于事实源。
- `wiki/sources/` 不等于 raw `sources/` bucket；它是所有 raw item 的 provenance / source page 层。
- wiki `knowledge/` 是 LLM 维护的结构化知识节点，不是普通个人笔记。
- source page 不能替代 knowledge page；knowledge page 也不能复制 raw 原文。

## 适用场景

当用户表达以下意图时使用：

- “把这个文件 ingest 到 wiki”
- “处理 raw/inbox 里的资料”
- “把这些来源整理进 raw/library”
- “给这个来源建 source page”
- “把这个来源重新 ingest”
- “原文更新了，同步 wiki”
- “处理一批待 ingest 文档”

如果用户要做的是：

- 初始化或补齐 wiki 骨架 -> 这不是本 skill
- 基于现有 wiki 回答问题 -> 这不是本 skill
- 巡检一致性、孤儿页、needs-update -> 这不是本 skill
- 只改写某个 wiki 页面且没有具体 raw 来源 -> 这不是本 skill

## 工作模式

默认支持三种模式，先读现状后判断：

1. **首次 ingest**
   - 来源在 `raw/inbox/<bucket>/`，或已放入 `raw/library/<bucket>/`
   - `raw-index.md` 中还没有稳定条目
   - 需要登记、建来源页、抽取知识点、创建或更新必要 knowledge pages，并建立双链

2. **re-ingest / update**
   - 来源已在 `raw/library/<bucket>/`
   - `raw-index.md` 已有条目
   - 用户要求刷新，或原文、页面结构、证据链发生变化
   - 需要保留 source identity，不重复制造来源页

3. **批量 ingest**
   - 用户要求处理多个文件或一个目录
   - 应逐个来源处理
   - 每个来源都要有独立 raw-index 状态、source page、必要 knowledge pages、关联建议与日志记录

## 核心原则

1. **先确认目标来源**
   - 如果用户点名文件或目录，以该路径为准。
   - 如果用户只说处理 inbox，先列出 `raw/inbox/sources/` 与 `raw/inbox/notes/` 中的候选。
   - 如果候选多个且用户没有授权批量处理，先确认范围。
   - 不要凭文件名猜测不存在的来源。

2. **raw 是事实源**
   - 可以移动或整理 raw 文件位置。
   - 不改写原始资料正文。
   - 不把提取结果塞回 raw 原文。
   - 不为了补 frontmatter 而改写 `raw/library/notes/` 中的原始笔记；frontmatter 属于生成或更新的 wiki 页面。
   - 不在 raw 下创建 `syntheses/`。

3. **bucket 只用 sources / notes**
   - 外部资料默认归 `sources`，例如文章、论文、网页、PDF、访谈、视频文字稿、书摘。
   - 用户自己的原始笔记、摘录、临时记录可归 `notes`。
   - 不按 articles、videos、books、webclips 等类型建目录。
   - 类型差异写进 source page 或 knowledge page 内部字段。

4. **按需创建目录**
   - 需要处理外部来源时，才创建 `raw/inbox/sources/`、`raw/library/sources/`、`wiki/sources/`。
   - 需要处理原始笔记时，才创建 `raw/inbox/notes/`、`raw/library/notes/`、`wiki/sources/`，必要时创建 `wiki/knowledge/`。
   - 只有形成跨来源综合时，才创建 `wiki/syntheses/`。
   - 创建目录是行为结果，不要求 init 预先创建。

5. **inbox 到 library 的处理对所有 bucket 一致**
   - 从 `raw/inbox/<bucket>/` 移动文件到 `raw/library/<bucket>/` 后，必须保留 `raw/inbox/<bucket>/`。
   - 即使 `raw/inbox/<bucket>/` 暂时为空，也不要删除。
   - 不删除 `raw/inbox/`、`raw/library/` 或任何用户已有目录。
   - `sources` 和 `notes` 都使用同一条规则：完成 ingest 后，已入库文件不应继续留在 `raw/inbox/<bucket>/` 中。
   - 如果 `raw/library/<bucket>/` 已经存在同名且内容相同的文件，应把 `raw/inbox/<bucket>/` 中的重复文件视为已入库副本并移除；目录本身仍保留。
   - 如果同名文件内容不同，不要覆盖或删除，先报告冲突并让用户决定保留哪个版本。

6. **raw-index 是状态中心**
   - 只有 `raw/library/` 中的资料登记到 `raw-index.md`。
   - `raw/inbox/` 中的文件表示待处理，不在 raw-index 中创建正式条目。
   - 状态只使用 `ingested`、`needs-update`。
   - 正常 ingest 完成后登记为 `ingested`。
   - `needs-update` 只能由用户明确要求或明确更新信号触发，并且必须有 `update_reason`。
   - 时间字段使用 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。

7. **Page Shapes 是页面结构来源**
   - 创建或更新 `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/` 时，遵守 `system/workflow.md` 中的 `Page Shapes`。
   - 如果 `system/workflow.md` 缺失或没有 `Page Shapes`，先补齐 workflow，再创建页面。
   - 页面必须使用 YAML frontmatter；frontmatter 用于 Obsidian 属性、检索和 Dataview，不替代正文证据与 `[[...]]` 双链。
   - 所有 wiki 页面都要维护 `page_type`、`tags`、`created_at`、`updated_at`。
   - `tags` 必须包含页面类型标签和从内容抽取的关键词；标签数量保持克制，通常 3-8 个关键词。
   - source page 使用 `llm-wiki/source`，knowledge page 使用 `llm-wiki/knowledge`，synthesis page 使用 `llm-wiki/synthesis`。

8. **必须建立 Obsidian 关联**
   - 目录位置不能替代页面关系。
   - source page 必须用 `[[...]]` 链接到它同步过或明确相关的 knowledge page / synthesis。
   - knowledge page 必须用 `[[...]]` 链接到关键 source page 和相关 knowledge page。
   - synthesis page 必须用 `[[...]]` 链接到相关 source page 和 knowledge page。
   - ingest 时必须查看已有 `wiki/knowledge/` 和 `wiki/syntheses/`，主动寻找可链接页面。
   - 如果已有页面相关，必须更新双链，不要只写“暂无”。
   - 如果当前来源抽取出了稳定方法论、概念、问题或操作原则，且没有合适 knowledge page，必须创建最小 knowledge page 并链接。
   - 只有在来源确实没有可沉淀知识点，且已有 wiki 中也没有相关页面时，才允许写空关联。

9. **必须做知识链更新**
   - source page 是 ingest 的必做产物。
   - 每个登记到 `raw-index.md` 的 raw item 都必须有对应 `wiki/sources/` 追踪页。
   - raw bucket 只决定原始资料放在 `sources` 还是 `notes`，不决定 wiki 侧是否抽取知识链。
   - `wiki/sources/` 是 provenance 层，不是 `raw/sources/` 的镜像；`raw/library/notes/` 也应先有对应 source page，再按内容抽取 knowledge。
   - 无论来源来自 `raw/library/sources/` 还是 `raw/library/notes/`，都要基于内容提取核心联系，创建或更新必要的 source page、knowledge page 和 synthesis 候选。
   - knowledge page 不是 query 的专属产物；ingest 应主动创建或更新用于承载方法论、概念、问题、操作原则的 knowledge page。
   - synthesis page 是高门槛产物，但 ingest 仍应基于当前来源和已有 wiki 判断是否有相关 synthesis 可链接或建议更新。
   - 优先更新已有页面，避免为弱信息批量制造碎片页。
   - 不确定是否值得新建 synthesis 时，把候选写入 source page 的“可能的 syntheses”，并链接已有相关 source/knowledge。

10. **同步要克制**
   - 主动同步不等于无限扩张。
   - 一个来源通常创建少量高价值 knowledge pages，而不是为每个名词建页。
   - knowledge pages 应围绕可复用的方法论、概念、问题、决策原则或操作经验。
   - 纯事实摘录、临时细节、一次性上下文留在 source page。

11. **不把猜测写成事实**
   - 明确区分结论、证据、摘录、推断、不确定和冲突。
   - 没有来源支撑的内容，只能写入“待跟进问题”或“不确定”。

## 目标结果

一次成功 ingest 后，通常应满足：

- 目标资料位于 `raw/library/<bucket>/`
- 原 `raw/inbox/<bucket>/` 目录仍然存在
- 已完成 ingest 的 inbox 文件不再留在 `raw/inbox/<bucket>/` 中
- `raw-index.md` 有对应条目
- 条目状态与时间合理；只有状态为 `needs-update` 时才填写 `update_reason`
- `wiki/sources/<source-page>.md` 已创建或更新
- 如果 raw item 来自 `raw/library/notes/`，也必须有对应 `wiki/sources/<source-page>.md`
- 必要的 `wiki/knowledge/` 已创建或更新
- 相关 `wiki/syntheses/` 已链接、更新或给出候选建议
- `index.md` 已更新高价值入口
- `log.md` 已追加记录

## 执行流程

source page 不是 ingest 的终点。执行流程必须完成“来源入库 -> 整体语料分析 -> source page -> knowledge page -> synthesis 候选/综合页 -> 双链验证”的闭环。

### 1. 检查基础 schema

先确认目标 vault 至少有：

- `CLAUDE.md`
- `raw-index.md`
- `index.md`
- `log.md`
- `raw/inbox/`
- `raw/library/`
- `wiki/`
- `system/workflow.md`
- `system/raw-index-rules.md`

如果缺少关键骨架，先提示应运行 `personal-wiki-init` 或补齐缺失项。不要在 ingest 中临时发明另一套 schema。

### 2. 确认来源与 bucket

读取目标来源前，先判断：

- 来源路径
- 当前位于 `raw/inbox/` 还是 `raw/library/`
- bucket 是 `sources` 还是 `notes`
- 是否已在 `raw-index.md` 登记
- 是否已有对应 source page
- 已有 `wiki/knowledge/` 中是否存在相关方法论、概念、问题或操作原则
- 已有 `wiki/syntheses/` 中是否存在相关综合页
- 是否是首次 ingest、re-ingest，还是批量 ingest

如果来源不在 `raw/inbox/` 或 `raw/library/`，先询问是否要复制或移动进当前 vault；不要直接把任意外部路径当作已入库来源。

### 3. 整理 raw 文件

如果来源在 `raw/inbox/<bucket>/`：

1. 确认或创建 `raw/library/<bucket>/`。
2. 默认将来源移动到 `raw/library/<bucket>/`；不要因为是 `notes` bucket 就只复制不移动。
3. 保留 `raw/inbox/<bucket>/` 目录。
4. 不删除空目录。
5. 不改写 raw 正文。

如果目标 `raw/library/<bucket>/` 已存在同名文件：

1. 先比较 inbox 文件与 library 文件。
2. 内容相同：使用 library 文件作为正式 `raw_path`，移除 inbox 中的重复文件，保留 inbox bucket 目录。
3. 内容不同：不要覆盖、不要删除，报告路径冲突，让用户决定是重命名、覆盖还是保留两个版本。

如果来源已经在 `raw/library/<bucket>/`：

1. 不重复制造副本。
2. 使用现有 library 路径作为 `raw_path`。
3. 进入 raw-index 与页面更新流程。

### 4. 更新 raw-index.md

`raw-index.md` 条目应与 init schema 保持一致，至少包含：

- `id`
- `title`
- `raw_path`
- `status`
- `last_status_at`
- `wiki_page`
- `update_reason`

状态规则：

- `ingested`：source page 与必要同步已完成。
- `needs-update`：用户明确要求更新，或存在明确更新信号，必须填写 `update_reason`。

处理建议：

- 首次 ingest 时，完成 source page 和必要同步后再登记或更新为 `ingested`。
- 如果用户明确说“需要更新”“重新 ingest”“原文更新了”，可以先判断是否需要标 `needs-update`；如果本次直接完成 re-ingest，最终状态写 `ingested`。
- 如果存在明确更新信号，例如 raw 文件修改时间晚于 `last_status_at`，可以使用 `needs-update` 并写明原因。
- 如果只是信息不足、证据不足或有未解问题，不写 `needs-update`；把问题写入 source page 的“待跟进问题”或 `log.md`。
- re-ingest 时保留原 `id` 和 `wiki_page`，只更新时间与状态。
- `update_reason` 不是 re-ingest 日志字段；当最终状态是 `ingested` 时，`update_reason` 应为空。
- re-ingest 的触发原因写入 `log.md`，例如 `manual-reingest`、`source-changed`、`link-refresh`。

### 5. 构建整体分析集合

在创建或更新 source page 之前，必须把本次来源与已有 wiki 文档放在一起分析。

分析集合至少包括：

- 本次明确 ingest 的 raw 来源内容。
- 该来源已有的 `wiki/sources/` 页面，如果存在。
- 已有 `wiki/sources/` 中与本次来源主题、概念、问题、方法相近的页面。
- 已有 `wiki/knowledge/` 中可能承载相同方法论、概念、问题、原则或经验的页面。
- 已有 `wiki/syntheses/` 中可能相关的综合页。

整体分析要回答：

- 本次来源补充了哪些已有 knowledge page。
- 本次来源是否暴露出新的可复用 knowledge page。
- 本次来源与哪些历史 source 形成同一问题链、概念链或方法论链。
- 本次来源是否改变、增强或冲突于已有 synthesis。
- 如果暂不创建 synthesis，候选综合方向是什么，相关证据链包括哪些 `[[source]]` / `[[knowledge]]`。

不允许只读当前来源就写“暂无关联”。只有在整体分析集合中确实找不到可沉淀知识点或历史关联时，才允许空关联。

### 6. 抽取 knowledge 与 synthesis 候选

从整体分析集合中抽取候选，而不是只从当前来源抽取。

knowledge 候选包括：

- 方法论
- 概念
- 问题
- 决策原则
- 操作经验
- 可复用判断框架

每个 knowledge 候选都要判断：

- 是否已有 `wiki/knowledge/` 页面可更新
- 是否需要创建新的最小 knowledge page
- 支撑证据来自本次来源还是历史 source
- 与哪些 source / knowledge / synthesis 建立 `[[...]]` 链接
- 应抽取哪些关键词写入页面 `tags`

synthesis 候选包括：

- 多个来源之间的共识
- 多个来源之间的分歧
- 阶段性结论
- 跨来源方法论链条
- 需要后续 query 深化的综合方向

如果存在候选 synthesis，但还不适合创建综合页，必须写入 source page 的“可能的 syntheses”，并附上相关 `[[source]]` / `[[knowledge]]` 链接。

关键词抽取规则：

- tags 是页面属性，不是正文分类目录。
- tags 必须来自当前来源和整体分析集合，不凭空扩展。
- tags 优先覆盖主题、概念、方法、问题、工具或领域，不收录过细的一次性名词。
- 英文多词标签使用短横线，例如 `knowledge-management`；中文标签不要包含空格。
- tags 不能替代 `[[...]]` 双链；页面关系仍必须写入正文链接。

### 7. 创建或更新 source page

source page 位于 `wiki/sources/`。如果目录不存在，在创建页面时再创建。

文件名应稳定、可读、避免重复。已有 source page 时优先更新，不另建相似页面。

页面结构遵守 `system/workflow.md` 的 `Source Page` 形状，至少覆盖：

- YAML frontmatter
- 基本信息
- 原始路径
- 摘要
- 关键观点
- 证据 / 摘录
- 关联 knowledge
- 可能的 syntheses
- 待跟进问题

写入要求：

- frontmatter 至少包含 `page_type: source`、`tags`、`created_at`、`updated_at`、`raw_path`、`raw_index_id`、`raw_bucket`。
- `raw_bucket` 只能是 `sources` 或 `notes`，必须与 `raw_path` 所在 bucket 一致。
- `tags` 至少包含 `llm-wiki/source` 和从来源内容抽取的关键词。
- 新建页面时写入 `created_at` 和 `updated_at`；更新页面时保留原 `created_at`，只刷新 `updated_at`。
- 摘要必须来自 raw 内容，不凭空补全。
- 关键观点要能回到证据或摘录。
- 人物、概念、主题、问题写入页面内部字段、标签或小节，不拆成目录。
- 不把 raw-index 当作知识提取仓库。
- 关联 knowledge 必须链接本次创建、更新或识别出的相关 knowledge page。
- 可能的 syntheses 不只看当前来源，也要结合已有 wiki/source/knowledge 判断。
- 如果已有 synthesis 相关，必须用 `[[...]]` 链接。
- 如果还不应该创建 synthesis，也要写出候选综合方向和相关 `[[source]]` / `[[knowledge]]` 证据链。
- 如果更新了 knowledge page 或 synthesis，也要在对应页面反向链接到本 source page。

source page 中不允许只写空的“关联 knowledge / 可能的 syntheses”，除非第 5 步整体分析和第 6 步候选抽取确认确实没有可沉淀知识点或历史关联。

### 8. 创建或更新 knowledge pages

ingest 必须尝试同步 `wiki/knowledge/`。只要来源中存在可复用的方法论、概念、问题、决策原则或操作经验，就应创建或更新 knowledge page。

同步规则：

- 已有相关 knowledge page：优先更新。
- 没有相关 knowledge page：如果信息足够稳定、可复用，创建最小 knowledge page。
- 新建或更新时遵守 `system/workflow.md` 的 `Knowledge Page` 形状。
- frontmatter 至少包含 `page_type: knowledge`、`tags`、`created_at`、`updated_at`、`knowledge_type`。
- `tags` 至少包含 `llm-wiki/knowledge` 和从该知识页承载内容中抽取的关键词。
- `knowledge_type` 用于标识页面承载类型，例如 `methodology`、`concept`、`question`、`principle`、`practice` 或 `framework`。
- 更新页面时保留原 `created_at`，刷新 `updated_at`，并按新增内容调整 tags。
- 每个新增观点都要保留来源链接或证据。
- 关键来源必须使用 `[[...]]` 链接到 source page。
- 相关 knowledge pages 必须使用 `[[...]]` 双链。
- 冲突、不确定和待查证内容要单独标注。

不要把一个来源拆成大量薄 knowledge pages。信息不足、一次性事实或临时上下文留在 source page 的“待跟进问题”。

不允许因为“当前没有现成 knowledge page”就停止关联；应先判断是否需要创建最小 knowledge page。

### 9. 判断 syntheses 关联

默认不强制新建 synthesis，但必须检查已有 `wiki/syntheses/`，并在 source page 中给出综合关联判断。

满足以下条件之一时，更新或创建 `wiki/syntheses/`：

- 本次来源改变了已有综合页的结论。
- 多个来源之间出现明确共识或分歧，需要记录。
- 用户明确要求形成跨来源综合。
- 当前来源与已有多个 sources / knowledge pages 形成稳定方法论链条。

新建或更新时遵守 `system/workflow.md` 的 `Synthesis Page` 形状，并明确：

- YAML frontmatter
- 范围
- 当前结论
- 共识
- 分歧
- 证据链
- 缺口
- 相关 sources / knowledge

frontmatter 至少包含 `page_type: synthesis`、`tags`、`created_at`、`updated_at`、`synthesis_type`。

`tags` 至少包含 `llm-wiki/synthesis` 和综合方向关键词。`synthesis_type` 用于标识综合类型，例如 `consensus`、`conflict`、`comparison`、`decision` 或 `research-thread`。

相关 sources / knowledge 必须使用 `[[...]]` 双链。

如果不创建 synthesis，也要在 source page 的“可能的 syntheses”中说明：

- 可综合的方向
- 相关已有 `[[source]]` / `[[knowledge]]`
- 为什么暂不创建独立 synthesis

这是 ingest 的必做流程，不是 query 的前置任务。query 可以深化综合，但 ingest 必须先给出基于本次来源和历史 wiki 的候选关联。

### 10. 更新 index.md

只更新高价值入口：

- 新增的重要 source page
- 值得追踪的 knowledge page
- 重要 synthesis
- 当前待处理问题或动作
- 本次 ingest / re-ingest / batch ingest 的完成记录，写入“最近新增”

不要把 `index.md` 变成全量文件清单。

维护规则：

- “待处理”只放尚未完成、需要后续 action 的事项。
- 已完成的 ingest、re-ingest、补链、frontmatter 修复、schema 对齐，不要写在“待处理”。
- 如果“待处理”里原有事项已经被本次 ingest 完成，应移出“待处理”，并在“最近新增”中追加完成记录。
- “最近新增”应使用真实当前本地时间，格式 `YYYY-MM-DD HH:MM:SS`，写入前必须读取时间。
- “最近新增”记录应简短说明动作类型、影响范围和关键页面，例如 source pages / knowledge / synthesis。
- 不要把 `log.md` 的完整日志复制到 `index.md`；`index.md` 只保留高价值摘要入口。

### 11. 追加 log.md

每个来源至少追加一条记录。批量 ingest 时，可以一条总记录加逐项列表。

记录至少包含：

- 时间 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间
- 操作类型：`ingest` 或 `re-ingest`
- re-ingest 的触发原因，如 `manual-reingest`、`source-changed`、`link-refresh`
- raw 路径
- source page
- 更新的 knowledge pages / syntheses
- raw-index 状态
- 如果 raw-index 状态是 `needs-update`，写明 `update_reason`

### 12. 最小验证

完成后检查：

- 来源是否位于 `raw/library/<bucket>/`
- 原 `raw/inbox/<bucket>/` 目录是否仍存在
- 已入库的 inbox 文件是否已移动走，或在 library 已有同内容副本时已移除重复 inbox 文件
- `raw-index.md` 是否有条目
- `status` 与 `last_status_at` 是否更新
- `needs-update` 是否有 `update_reason`
- `ingested` 状态是否没有滥填 `update_reason`
- source page 是否存在并符合 Page Shapes
- source page 是否有合法 frontmatter、页面类型标签和抽取关键词 tags
- source page 是否包含 `raw_bucket`，并且 `raw_bucket` 与 raw 路径一致
- `raw/library/notes/` 中已 ingest 的文件是否也有对应 `wiki/sources/` 追踪页
- 无论处理的是 `sources` 还是 `notes`，对应 wiki source page / knowledge page 是否仍按 Page Shapes 写入 frontmatter；不要把 frontmatter 补到 raw 原文里
- 是否构建了本次来源 + 历史 wiki 的整体分析集合
- 是否从整体分析集合抽取 knowledge 候选和 synthesis 候选
- source page 与同步过的 knowledge pages / syntheses 是否存在 `[[...]]` 链接
- knowledge pages / syntheses 是否反向链接到关键 source page
- knowledge pages / syntheses 是否维护对应 `page_type`、类型标签、抽取关键词和更新时间
- 是否主动检查并链接已有相关 knowledge pages
- 当前来源有稳定方法论、概念、问题或原则时，是否创建或更新 knowledge page
- 是否检查已有 syntheses，并在 source page 写出候选综合方向
- `index.md` 是否只更新高价值入口
- `index.md` 的“待处理”是否没有保留本次已完成事项
- `index.md` 的“最近新增”是否追加本次 ingest / re-ingest / batch ingest 摘要
- `log.md` 是否追加记录

## 汇报格式

默认用简洁中文汇报：

```md
已完成 ingest（首次 ingest / re-ingest / 批量 ingest）。

处理来源：
- ...

已更新：
- raw/library/...
- raw-index.md
- wiki/sources/...
- wiki/knowledge/...（如有）
- wiki/syntheses/...（如有）
- index.md
- log.md

状态：
- status: ingested / needs-update
- update_reason: ...（仅 needs-update 时填写）

标签：
- wiki/sources/...: ...
- wiki/knowledge/...: ...（如有）
- wiki/syntheses/...: ...（如有）

注意点：
- ...
```

如果没有创建 knowledge page，要明确说明为什么当前来源没有可复用的方法论、概念、问题或原则。不能只说“没有已有目标页”。

如果没有创建 syntheses，要说明是否检查了历史 source/knowledge，以及候选综合方向是什么。

## 不要这样做

- 不要跳过 `raw-index.md`
- 不要登记仍停留在 `raw/inbox/` 的临时文件
- 不要删除 `raw/inbox/` 或 `raw/inbox/<bucket>/`
- 不要在 raw 下创建 `syntheses/`
- 不要创建 `articles/`、`videos/`、`books/`、`webclips/` 等 raw 类型目录
- 不要创建 `topics/`、`people/`、`concepts/`、`questions/` 等 wiki 目录
- 不要把所有来源都强行拆成 knowledge page 或 synthesis
- 不要因为没有现成 knowledge page 就放弃创建必要 knowledge page
- 不要把可能的 syntheses 只局限于当前来源
- 不要用 re-ingest 制造重复 source page
- 不要把 query 或 lint 的职责混进 ingest
- 不要把猜测写成事实

## 示例触发

**示例 1**
用户：`把 raw/inbox/sources 里的这篇文章 ingest 到 wiki。`

动作：使用本 skill，移动到 `raw/library/sources/`，保留 `raw/inbox/sources/`，更新 raw-index、source page、必要 knowledge pages、index 和 log。

**示例 2**
用户：`这个来源已经在 library 里了，帮我重新 ingest。`

动作：保留原 source identity，更新 raw-index 状态、source page 和必要下游页面，不重复创建来源页。

**示例 3**
用户：`处理 raw/inbox/notes 里的这些原始笔记。`

动作：按统一 inbox -> library 规则处理，默认移动到 `raw/library/notes/`，只保留 `raw/inbox/notes/` 目录，并按 Page Shapes 创建或更新带 frontmatter 的 `wiki/sources/` / `wiki/knowledge/` 页面。
