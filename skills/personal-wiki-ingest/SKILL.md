---
name: personal-wiki-ingest
description: 当用户要把一份或一批原始资料纳入个人 LLM Wiki 的正式知识流时使用这个 skill。适用于处理 raw/inbox 或 raw/library 中的来源、把资料整理进 library、登记 raw-index、创建或更新 wiki source page、同步必要 notes、重新 ingest 已有来源。不要用于初始化骨架、纯查询回答、lint 巡检、或不围绕具体 raw 来源的普通页面编辑。
---

# Personal Wiki Ingest

这个 skill 用于把 `raw/inbox/` 或 `raw/library/` 中的具体资料纳入正式知识流。

ingest 的核心不是“搬文件”，而是完成一条可追踪链路：

```text
raw/inbox/ 或 raw/library/
-> raw/library/
-> raw-index.md
-> wiki/sources/
-> 必要的 wiki/notes/ 或 wiki/syntheses/
-> index.md
-> log.md
```

本 skill 以 `personal-wiki-init` 定义的最小 schema 为准：

- raw 只按需使用 `sources/`、`notes/`
- wiki 只按需使用 `sources/`、`notes/`、`syntheses/`
- `raw-index.md` 是唯一官方 raw ingest 状态表
- 页面结构来自 `system/workflow.md` 的 `Page Shapes`
- 不使用 `topics/`、`people/`、`concepts/`、`questions/` 作为目录
- wiki 页面之间的关系必须写成 Obsidian `[[...]]` 双链

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
   - 需要登记、建来源页、同步必要知识页

2. **re-ingest / update**
   - 来源已在 `raw/library/<bucket>/`
   - `raw-index.md` 已有条目
   - 用户要求刷新，或原文、页面结构、证据链发生变化
   - 需要保留 source identity，不重复制造来源页

3. **批量 ingest**
   - 用户要求处理多个文件或一个目录
   - 应逐个来源处理
   - 每个来源都要有独立 raw-index 状态、source page 与日志记录

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
   - 不在 raw 下创建 `syntheses/`。

3. **bucket 只用 sources / notes**
   - 外部资料默认归 `sources`，例如文章、论文、网页、PDF、访谈、视频文字稿、书摘。
   - 用户自己的原始笔记、摘录、临时记录可归 `notes`。
   - 不按 articles、videos、books、webclips 等类型建目录。
   - 类型差异写进 source page 或 note page 内部字段。

4. **按需创建目录**
   - 需要处理外部来源时，才创建 `raw/inbox/sources/`、`raw/library/sources/`、`wiki/sources/`。
   - 需要处理原始笔记时，才创建 `raw/inbox/notes/`、`raw/library/notes/`，必要时创建 `wiki/notes/`。
   - 只有形成跨来源综合时，才创建 `wiki/syntheses/`。
   - 创建目录是行为结果，不要求 init 预先创建。

5. **不删除 raw/inbox 目录**
   - 从 `raw/inbox/<bucket>/` 移动文件到 `raw/library/<bucket>/` 后，必须保留 `raw/inbox/<bucket>/`。
   - 即使 `raw/inbox/<bucket>/` 暂时为空，也不要删除。
   - 不删除 `raw/inbox/`、`raw/library/` 或任何用户已有目录。

6. **raw-index 是状态中心**
   - 只有 `raw/library/` 中的资料登记到 `raw-index.md`。
   - `raw/inbox/` 中的文件表示待处理，不在 raw-index 中创建正式条目。
   - 状态只使用 `ingested`、`needs-update`。
   - 正常 ingest 完成后登记为 `ingested`。
   - `needs-update` 只能由用户明确要求或明确更新信号触发，并且必须有 `update_reason`。
   - 时间字段使用 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。

7. **Page Shapes 是页面结构来源**
   - 创建或更新 `wiki/sources/`、`wiki/notes/`、`wiki/syntheses/` 时，遵守 `system/workflow.md` 中的 `Page Shapes`。
   - 如果 `system/workflow.md` 缺失或没有 `Page Shapes`，先补齐 workflow，再创建页面。

8. **必须建立 Obsidian 关联**
   - 目录位置不能替代页面关系。
   - source page 必须用 `[[...]]` 链接到它同步过或明确相关的 note / synthesis。
   - note page 必须用 `[[...]]` 链接到关键 source page 和相关 note。
   - synthesis page 必须用 `[[...]]` 链接到相关 source page 和 note。
   - 如果只创建了 source page，没有足够信息创建 note，也要在 source page 中保留原始路径和可验证证据；不要伪造链接。

9. **同步要克制**
   - source page 是 ingest 的必做产物。
   - note page 是条件产物：只有来源提供了值得长期维护的知识点时才创建或更新。
   - synthesis page 是高门槛产物：只有跨来源比较、阶段性综合或稳定结论被本次 ingest 改变时才创建或更新。
   - 优先更新已有页面，避免为弱信息批量制造碎片页。

10. **不把猜测写成事实**
   - 明确区分结论、证据、摘录、推断、不确定和冲突。
   - 没有来源支撑的内容，只能写入“待跟进问题”或“不确定”。

## 目标结果

一次成功 ingest 后，通常应满足：

- 目标资料位于 `raw/library/<bucket>/`
- 原 `raw/inbox/<bucket>/` 目录仍然存在
- `raw-index.md` 有对应条目
- 条目状态、时间、`update_reason` 合理
- `wiki/sources/<source-page>.md` 已创建或更新
- 必要的 `wiki/notes/` 或 `wiki/syntheses/` 已创建或更新
- `index.md` 已更新高价值入口
- `log.md` 已追加记录

## 执行流程

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
- 是否是首次 ingest、re-ingest，还是批量 ingest

如果来源不在 `raw/inbox/` 或 `raw/library/`，先询问是否要复制或移动进当前 vault；不要直接把任意外部路径当作已入库来源。

### 3. 整理 raw 文件

如果来源在 `raw/inbox/<bucket>/`：

1. 确认或创建 `raw/library/<bucket>/`。
2. 将来源移动到 `raw/library/<bucket>/`，或在用户要求保留 inbox 原件时复制到 library。
3. 保留 `raw/inbox/<bucket>/` 目录。
4. 不删除空目录。
5. 不改写 raw 正文。

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
- 如果用户明确说“需要更新”“重新 ingest”“原文更新了”，使用 `needs-update` 或直接执行 re-ingest 后改回 `ingested`。
- 如果存在明确更新信号，例如 raw 文件修改时间晚于 `last_status_at`，可以使用 `needs-update` 并写明原因。
- 如果只是信息不足、证据不足或有未解问题，不写 `needs-update`；把问题写入 source page 的“待跟进问题”或 `log.md`。
- re-ingest 时保留原 `id` 和 `wiki_page`，只更新时间与状态。

### 5. 创建或更新 source page

source page 位于 `wiki/sources/`。如果目录不存在，在创建页面时再创建。

文件名应稳定、可读、避免重复。已有 source page 时优先更新，不另建相似页面。

页面结构遵守 `system/workflow.md` 的 `Source Page` 形状，至少覆盖：

- 基本信息
- 原始路径
- 摘要
- 关键观点
- 证据 / 摘录
- 关联 notes
- 可能的 syntheses
- 待跟进问题

写入要求：

- 摘要必须来自 raw 内容，不凭空补全。
- 关键观点要能回到证据或摘录。
- 人物、概念、主题、问题写入页面内部字段、标签或小节，不拆成目录。
- 不把 raw-index 当作知识提取仓库。
- 关联 notes 和可能的 syntheses 必须使用 `[[...]]` 双链。
- 如果更新了 note 或 synthesis，也要在对应页面反向链接到本 source page。

### 6. 同步 notes

只有在来源提供了值得长期维护的知识点时，才同步 `wiki/notes/`。

同步规则：

- 已有相关 note：优先更新。
- 没有相关 note：只有信息足够稳定、可复用，才新建。
- 新建或更新时遵守 `system/workflow.md` 的 `Note Page` 形状。
- 每个新增观点都要保留来源链接或证据。
- 关键来源必须使用 `[[...]]` 链接到 source page。
- 相关 notes 必须使用 `[[...]]` 双链。
- 冲突、不确定和待查证内容要单独标注。

不要把一个来源拆成大量薄 note。信息不足时留在 source page 的“待跟进问题”。

### 7. 同步 syntheses

默认不新建 synthesis。只有满足以下条件之一才处理 `wiki/syntheses/`：

- 本次来源改变了已有综合页的结论。
- 多个来源之间出现明确共识或分歧，需要记录。
- 用户明确要求形成跨来源综合。

新建或更新时遵守 `system/workflow.md` 的 `Synthesis Page` 形状，并明确：

- 范围
- 当前结论
- 共识
- 分歧
- 证据链
- 缺口
- 相关 sources / notes

相关 sources / notes 必须使用 `[[...]]` 双链。

### 8. 更新 index.md

只更新高价值入口：

- 新增的重要 source page
- 值得追踪的 note
- 重要 synthesis
- 当前待处理问题或动作

不要把 `index.md` 变成全量文件清单。

### 9. 追加 log.md

每个来源至少追加一条记录。批量 ingest 时，可以一条总记录加逐项列表。

记录至少包含：

- 时间 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间
- 操作类型：`ingest` 或 `re-ingest`
- raw 路径
- source page
- 更新的 notes / syntheses
- raw-index 状态
- 如果是 `needs-update`，写明 `update_reason`

### 10. 最小验证

完成后检查：

- 来源是否位于 `raw/library/<bucket>/`
- 原 `raw/inbox/<bucket>/` 目录是否仍存在
- `raw-index.md` 是否有条目
- `status` 与 `last_status_at` 是否更新
- `needs-update` 是否有 `update_reason`
- source page 是否存在并符合 Page Shapes
- source page 与同步过的 notes / syntheses 是否存在 `[[...]]` 链接
- notes / syntheses 是否反向链接到关键 source page
- 必要 notes / syntheses 是否同步
- `index.md` 是否只更新高价值入口
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
- wiki/notes/...（如有）
- wiki/syntheses/...（如有）
- index.md
- log.md

状态：
- status: ingested / needs-update
- update_reason: ...

注意点：
- ...
```

如果没有创建 notes 或 syntheses，要明确说明原因，例如“本来源信息不足以沉淀长期 note”。

## 不要这样做

- 不要跳过 `raw-index.md`
- 不要登记仍停留在 `raw/inbox/` 的临时文件
- 不要删除 `raw/inbox/` 或 `raw/inbox/<bucket>/`
- 不要在 raw 下创建 `syntheses/`
- 不要创建 `articles/`、`videos/`、`books/`、`webclips/` 等 raw 类型目录
- 不要创建 `topics/`、`people/`、`concepts/`、`questions/` 等 wiki 目录
- 不要把所有来源都强行拆成 note 或 synthesis
- 不要用 re-ingest 制造重复 source page
- 不要把 query 或 lint 的职责混进 ingest
- 不要把猜测写成事实

## 示例触发

**示例 1**
用户：`把 raw/inbox/sources 里的这篇文章 ingest 到 wiki。`

动作：使用本 skill，移动到 `raw/library/sources/`，保留 `raw/inbox/sources/`，更新 raw-index、source page、必要 notes、index 和 log。

**示例 2**
用户：`这个来源已经在 library 里了，帮我重新 ingest。`

动作：保留原 source identity，更新 raw-index 状态、source page 和必要下游页面，不重复创建来源页。

**示例 3**
用户：`处理 raw/inbox/notes 里的这些原始笔记。`

动作：按 `notes` bucket 批量处理，移动或复制到 `raw/library/notes/`，保留 `raw/inbox/notes/`，按需要更新 `wiki/notes/`。
