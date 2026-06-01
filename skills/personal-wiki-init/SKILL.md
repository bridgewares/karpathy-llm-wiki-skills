---
name: personal-wiki-init
description: 初始化或补齐个人 LLM Wiki / Obsidian-first 知识库骨架。只要用户是在说新建个人 wiki、初始化 vault、按 llm-wiki 思路搭目录、补齐缺失的 CLAUDE.md/raw-index.md/system 规则文件、或把半成品知识库补成可运行的最小结构，就应该主动使用这个 skill，即使用户没有明确说“skill”或“init”。仅用于骨架初始化与缺失补齐；如果用户是在导入来源、执行 ingest、回答知识问题、检查孤儿页或做 lint，则不要触发这个 skill。
---

# Personal Wiki Init

这个 skill 用于在一个目录中初始化或补齐个人 LLM Wiki 的最小可用骨架。

目标是建立后续 ingest、query、lint 必需的稳定入口和规则文件。初始化阶段不创建具体知识页，也不预建 bucket 子目录。

## 适用场景

当用户表达以下意图时使用：

- “帮我初始化个人 wiki”
- “按 llm-wiki 思路搭一个 vault”
- “给我的知识库建最小目录和规则文件”
- “把这个半成品 wiki 补齐到能继续 ingest/query/lint”
- “检查缺什么并初始化”

如果用户要做的是：

- 导入具体来源 -> 这不是本 skill
- 从 wiki 回答问题 -> 这不是本 skill
- 检查一致性、孤儿页、needs-update -> 这不是本 skill
- 改写已有知识页内容 -> 这不是本 skill

## 工作模式

本 skill 支持两种模式，默认先观察目标目录状态后自动判断：

1. **全新初始化**
   - 目标目录还没有可运行的 wiki 骨架。
   - 创建最小目录、根文件和 system 规则文件。

2. **缺失补齐**
   - 目标目录已经存在部分结构或部分文件。
   - 只补齐缺失项。
   - 对已有文件优先保持最小修改；除非用户明确要求，不整篇重写用户内容。
   - 如果已有规则文件包含旧 schema 冲突，应报告并做最小补丁。

3. **schema 冲突修正**
   - 目标目录已有 `CLAUDE.md`、`system/workflow.md` 或 `system/raw-index-rules.md`，但其中仍使用旧规则。
   - 只修正规则文件中的明确 schema 冲突。
   - 不迁移历史 wiki 页面、历史链接或历史目录。

## 核心原则

1. **先确认目标 vault 根目录**
   - 如果用户点名目录，以该目录为准。
   - 如果用户没有点名目录，默认以当前工作目录作为目标 vault 根目录。

2. **先读现状，再写内容**
   - 先检查已有文件与目录。
   - 不盲目覆盖用户已有内容。
   - 不删除用户已有内容。
   - 对已有规则文件中的 schema 冲突做最小修正，不整篇刷新。

3. **只创建最小骨架**
   - 初始化只建立稳定入口和后续必需规则。
   - 不导入真实来源。
   - 不创建具体 source page、knowledge page 或 synthesis page。
   - 不虚构 raw-index 条目。
   - 不自动创建大量空知识页。

4. **明确 raw / wiki / schema 边界**
   - `raw/` 保存原始资料，是事实源。
   - `wiki/` 保存 LLM 维护的结构化知识页。
   - `CLAUDE.md` 与 `system/` 定义 schema、工作流和页面结构。
   - `raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。

5. **目录按行为触发创建**
   - init 阶段只创建 `raw/inbox/`、`raw/library/`、`wiki/`。
   - `raw/inbox/` 和 `raw/library/` 下只允许按需创建 `sources/`、`notes/`。
   - `wiki/` 下只允许按需创建 `sources/`、`knowledge/`、`syntheses/`。
   - `notes/` 只属于 raw；wiki 第二桶必须命名为 `knowledge/`，不要创建或引用 `wiki/notes/`。
   - `syntheses/` 只属于 `wiki/`，不要在 raw 下创建。
   - 主题、人物、概念、问题等细分应作为页面内部类型、标签或 frontmatter，而不是目录名。

6. **页面结构写入 workflow**
   - `system/workflow.md` 必须包含 `Page Shapes`。
   - ingest/query 创建页面时，以 `system/workflow.md` 的 `Page Shapes` 为准。
   - 如果 `system/workflow.md` 缺失或没有 `Page Shapes`，后续创建页面前应先补齐 workflow。

7. **规则文件以最新 schema 为准**
   - 已有 `CLAUDE.md`、`system/workflow.md`、`system/raw-index-rules.md` 不是只读历史材料。
   - 如果它们仍定义 `wiki/notes/`、`Note Page`、`source、note、synthesis` 等旧 schema，应做最小文本补丁。
   - 最小补丁只限规则层：目录模型、Page Shapes、工作流步骤、Obsidian 链接说明、raw-index 规则。
   - 不在 init 中移动、重命名或删除已有 `wiki/notes/` 目录和历史页面；这类迁移交给 lint 或用户明确授权后的专项任务。

8. **中文优先**
   - 文档主体优先中文。
   - 保留必要英文术语，如 ingest、query、lint、source page、note、synthesis。

## 标准目标结构

初始化完成后，目标结构应至少包括：

```text
CLAUDE.md
raw-index.md
index.md
log.md
raw/
  inbox/
  library/
wiki/
system/
  workflow.md
  raw-index-rules.md
```

目录职责：

| 路径 | 职责 |
| --- | --- |
| `raw/inbox/` | 待处理原始资料入口。这里的文件还没有进入正式知识流，不登记到 `raw-index.md`。 |
| `raw/library/` | 正式原始资料库。只有这里的资料才登记到 `raw-index.md`。 |
| `raw/inbox/sources/` / `raw/library/sources/` | 外部来源原文，如文章、网页、论文、PDF、访谈、视频文字稿、书摘。 |
| `raw/inbox/notes/` / `raw/library/notes/` | 用户自己的原始笔记、摘录、临时记录。这里的 notes 是 raw 笔记，不是 wiki 知识页。 |
| `wiki/sources/` | 来源追踪页。每页对应一个 raw item，回答“这个 raw item 说了什么”，包括 `raw/library/sources/` 和 `raw/library/notes/`。 |
| `wiki/knowledge/` | 结构化知识页。承载可复用的方法论、概念、问题、决策原则或操作经验。 |
| `wiki/syntheses/` | 综合页。承载跨来源、跨知识页的比较、共识、分歧和阶段性结论。 |
| `system/` | 工作流、页面结构和 raw-index 规则。 |

Schema 约束：

```text
允许创建：
- system/workflow.md
- system/raw-index-rules.md

不创建：
- templates/
- system/taxonomy.md
- system/naming-conventions.md
```

不要在 init 阶段预建以下目录：

```text
raw/inbox/sources/
raw/inbox/notes/
raw/library/sources/
raw/library/notes/
wiki/sources/
wiki/knowledge/
wiki/syntheses/
```

这些目录由后续行为触发：

- ingest 外部来源时，创建 `raw/inbox/sources/`、`raw/library/sources/`、`wiki/sources/`
- ingest 用户原始笔记时，创建 `raw/inbox/notes/`、`raw/library/notes/`、`wiki/sources/`，必要时写入 `wiki/knowledge/`
- query 写回结构化知识页时，创建 `wiki/knowledge/`
- query 写回跨来源综合时，创建 `wiki/syntheses/`

## 初始化流程

按下面顺序执行。

### 1. 识别当前状态

先检查：

- 当前工作目录是否就是目标 vault 根目录
- 如果用户明确给了路径，是否应以该路径作为目标 vault 根目录
- 根目录是否已有 `CLAUDE.md`
- 是否已有 `raw/`、`wiki/`、`system/`
- 是否已有 `raw-index.md`、`index.md`、`log.md`
- `raw/` 是否已有 `inbox/` 与 `library/`
- `system/workflow.md` 是否存在并包含 `Page Shapes`
- `system/workflow.md` 是否定义 wiki 页面 frontmatter 与 tags 规则
- `system/raw-index-rules.md` 是否存在
- 现有规则文件中是否包含旧 schema，例如 `wiki/notes/`、`Note Page`、`关联 notes`、`source、note、synthesis`
- 是否存在历史 `wiki/notes/` 目录；如果存在，只报告，不迁移

然后判断：

- 关键结构几乎都不存在 -> 视为全新初始化
- 已有部分结构 -> 视为缺失补齐
- 已有规则文件存在但包含旧 schema -> 视为 schema 冲突修正

### 2. 创建或补齐目录

默认只确保以下目录存在：

- `raw/inbox/`
- `raw/library/`
- `wiki/`
- `system/`

不要因为“以后可能需要”而创建 bucket 子目录。

### 3. 创建或补齐根文件

确保以下文件存在：

- `CLAUDE.md`
- `raw-index.md`
- `index.md`
- `log.md`

#### `CLAUDE.md` 要点

必须清楚定义：

- 这是个人 LLM 维护的 wiki
- `raw/` 与 `wiki/` 的职责边界
- `raw/inbox/` 与 `raw/library/` 的区别
- raw 只使用 `sources`、`notes` 两类 bucket
- wiki 只使用 `sources`、`knowledge`、`syntheses` 三类 bucket
- `notes/` 只属于 raw，wiki 中不要使用 `notes/` 作为结构化知识页目录
- `raw-index.md` 是唯一官方 ingest 状态表
- `system/workflow.md` 是页面结构与工作流规则来源
- ingest/query/lint 工作流
- `index.md` 与 `log.md` 的维护要求
- 中文优先与秒级时间格式要求

#### `raw-index.md` 要点

必须清楚定义：

- 它是唯一官方 raw ingest 状态表
- 当前阶段暂不使用单文件 metadata 代替它
- 只有 `raw/library/` 中的来源才登记
- raw bucket 只能是 `sources` 或 `notes`
- 状态只使用：`ingested`、`needs-update`
- `needs-update` 只能由用户明确要求或明确更新信号触发，并且必须带 `update_reason`
- `ingested` 状态下 `update_reason` 应为空；re-ingest 原因记录在 `log.md`
- 所有状态时间字段使用 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间
- 可以提供按 `sources`、`notes` 分区的空白列表骨架

#### `index.md` 要点

必须包含：

- 当前重点入口
- Raw 管理入口
- Source pages 入口
- Knowledge / syntheses 入口
- 待处理问题或动作
- 最近新增入口

不要把 `index.md` 初始化成完整文件清单。

#### `log.md` 要点

初始化后至少要有一条初始化记录，说明：

- 已创建或补齐最小基础结构
- 已初始化 `raw-index.md`
- 已初始化 `system/workflow.md` 和 `system/raw-index-rules.md`
- 下一步建议是导入第一个真实来源并测试 ingest
- 记录时间必须是执行初始化时读取到的本地当前时间，不能使用占位符、日期零点或估计时间

### 4. 创建或补齐 system 规则文件

只生成 `system/workflow.md` 和 `system/raw-index-rules.md`。不要创建 schema 之外的 system 规则文件。

如果文件已存在，不要整篇覆盖；先判断是否缺少必需段落或存在旧 schema 冲突，再做最小补丁。

允许自动修正的规则层冲突包括：

- `wiki/notes/` -> `wiki/knowledge/`
- `Note Page` -> `Knowledge Page`
- `关联 notes` -> `关联 knowledge`
- `source、note、synthesis` -> `source、knowledge、synthesis`
- wiki 目录模型从 `sources/notes/syntheses` 改为 `sources/knowledge/syntheses`

不允许 init 自动执行的历史迁移包括：

- 移动或重命名已有 `wiki/notes/` 目录
- 批量改写历史页面中的 Obsidian 链接
- 删除旧页面或旧目录
- 把历史 note page 自动改写成 knowledge page

#### `system/workflow.md` 要点

必须覆盖：

- 目录模型：raw 使用 `sources/notes`，wiki 使用 `sources/knowledge/syntheses`
- ingest/query/lint 工作流
- Obsidian 双链规则
- 命名规范与时间格式
- 页面 frontmatter 规则
- 页面结构规则 `Page Shapes`

`Page Shapes` 是页面生成的唯一结构来源，至少包含：

```md
## Page Shapes

### Frontmatter Rules

所有 wiki 页面必须使用 YAML frontmatter。frontmatter 用于 Obsidian 属性、检索和 Dataview，不替代正文证据与 `[[...]]` 双链。

通用字段：
- `page_type`：`source`、`knowledge` 或 `synthesis`
- `tags`：包含页面类型标签和 3-8 个从内容抽取的关键词
- `created_at`：首次创建时间，格式 `YYYY-MM-DD HH:MM:SS`
- `updated_at`：最近更新时间，格式 `YYYY-MM-DD HH:MM:SS`

类型标签：
- source page 使用 `llm-wiki/source`
- knowledge page 使用 `llm-wiki/knowledge`
- synthesis page 使用 `llm-wiki/synthesis`

source page 额外字段：
- `raw_bucket`：`sources` 或 `notes`，必须与 `raw_path` 所在 bucket 一致

### Source Page

用于 `wiki/sources/`，由 ingest 创建或更新。

必须包含：
- YAML frontmatter：`page_type: source`、`tags`、`created_at`、`updated_at`、`raw_path`、`raw_index_id`、`raw_bucket`
- 基本信息
- 原始路径
- 摘要
- 关键观点
- 证据 / 摘录
- 关联 knowledge，使用 `[[...]]` 双链
- 可能的 syntheses，使用 `[[...]]` 双链
- 待跟进问题

### Knowledge Page

用于 `wiki/knowledge/`，由 ingest 或 query 创建或更新。

必须包含：
- YAML frontmatter：`page_type: knowledge`、`tags`、`created_at`、`updated_at`、`knowledge_type`
- 当前理解
- 关键来源
- 证据
- 相关 knowledge，使用 `[[...]]` 双链
- 不确定或冲突
- 下一步

### Synthesis Page

用于 `wiki/syntheses/`，由 query 或高价值 ingest 创建或更新。

必须包含：
- YAML frontmatter：`page_type: synthesis`、`tags`、`created_at`、`updated_at`、`synthesis_type`
- 范围
- 当前结论
- 共识
- 分歧
- 证据链
- 缺口
- 相关 sources / knowledge，使用 `[[...]]` 双链
```

#### `system/raw-index-rules.md` 要点

必须覆盖：

- `raw-index.md` 的职责边界
- raw bucket 只能是 `sources` 或 `notes`
- 状态语义：`ingested`、`needs-update`
- `needs-update` 只能由用户明确要求或明确更新信号触发，并且必须带 `update_reason`
- `ingested` 状态下 `update_reason` 应为空；re-ingest 原因记录在 `log.md`
- 必填字段
- 时间格式 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间

### 5. 文件不存在时的默认内容

如果目标文件不存在，按下面内容创建。可以根据目标 vault 名称做轻微替换，但不要增加 schema 之外的文件、目录或虚构来源。

#### `CLAUDE.md`

```md
# Personal LLM Wiki

这是一个由 LLM 辅助维护的个人 wiki。目标是把原始资料、结构化来源页、知识页与综合页分层管理，减少重复整理和无依据生成。

## 目录边界

- `raw/inbox/`：待处理原始资料入口。
- `raw/library/`：已纳入正式知识流的原始资料库，是事实源。
- `wiki/`：LLM 维护的结构化知识层。
- `system/`：工作流、页面结构与状态规则。

raw 下只按需使用：

- `sources/`
- `notes/`

wiki 下只按需使用：

- `sources/`
- `knowledge/`
- `syntheses/`

`notes/` 只属于 raw，用于用户原始笔记、摘录或临时记录；wiki 中的结构化知识页统一放在 `knowledge/`。

不要创建 `topics/`、`people/`、`concepts/`、`questions/` 等默认目录。主题、人物、概念和问题应写入页面 frontmatter、标签或正文小节。

## 状态与规则

- `raw-index.md` 是唯一官方 raw ingest 状态表。
- `system/workflow.md` 定义 ingest、query、lint 工作流和 Page Shapes。
- `system/raw-index-rules.md` 定义 raw-index 字段、状态和时间格式。
- 时间格式统一为 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。
- 文档主体中文优先，必要英文术语可以保留。

## 工作流

- init：只创建最小骨架和规则文件。
- ingest：把 `raw/inbox/` 或 `raw/library/` 中的具体资料纳入知识流，维护 `raw-index.md`，按需创建 wiki 页面。
- query：优先基于 `wiki/` 回答；证据不足时回到 `raw/library/` 补查；用户确认写回时复用已确认答案。
- lint：巡检 raw、raw-index、wiki 之间的一致性、结构与证据缺口。
```

#### `raw-index.md`

```md
# Raw Index

`raw-index.md` 是唯一官方 raw ingest 状态表。只有 `raw/library/` 中的资料登记在这里。

## 规则

- raw bucket 只能是 `sources` 或 `notes`。
- 状态只使用 `ingested`、`needs-update`。
- `needs-update` 只能由用户明确要求或明确更新信号触发，并且必须填写 `update_reason`。
- `ingested` 状态下 `update_reason` 留空；re-ingest 原因记录在 `log.md`。
- 时间字段使用 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。
- 不在这里登记虚构来源。

## Sources

| id | title | raw_path | status | last_status_at | wiki_page | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## Notes

| id | title | raw_path | status | last_status_at | wiki_page | update_reason |
| --- | --- | --- | --- | --- | --- | --- |
```

#### `index.md`

```md
# Wiki Index

## 当前重点

- 待补充

## Raw 管理

- `raw/inbox/`
- `raw/library/`
- `raw-index.md`

## Source Pages

- 待 ingest 后补充

## Knowledge / Syntheses

- 待 query 或 ingest 后补充

## 待处理

- 待处理来源以 `raw/inbox/` 中的真实文件为准。
- 待整理问题或动作可以临时记录在这里。

## 最近新增

- 待补充
```

#### `log.md`

创建时写入一条使用当前本地时间的记录。写入前必须先读取当前本地时间，格式为 `YYYY-MM-DD HH:MM:SS`。

不要使用：

- `YYYY-MM-DD HH:MM:SS`
- 未经读取的估计时间

```md
# Wiki Log

## <执行初始化时的本地当前时间>

- 初始化或补齐个人 wiki 最小骨架。
- 创建或确认 `raw/inbox/`、`raw/library/`、`wiki/`、`system/`。
- 初始化或确认 `raw-index.md`。
- 初始化或确认 `system/workflow.md` 和 `system/raw-index-rules.md`。
- 下一步：导入第一个真实来源并测试 ingest。
```

#### `system/workflow.md`

```md
# Workflow

## Directory Model

- init 只创建 `raw/inbox/`、`raw/library/`、`wiki/`、`system/`。
- raw 下只按需创建 `sources/`、`notes/`。
- wiki 下只按需创建 `sources/`、`knowledge/`、`syntheses/`。
- `syntheses/` 只属于 wiki，不属于 raw。

## Obsidian Links

- wiki 页面之间的关联必须写成 Obsidian 双链 `[[...]]`。
- source page 指向相关 knowledge 或 synthesis。
- knowledge page 指向关键 source 和相关 knowledge。
- synthesis page 指向相关 source 和 knowledge。
- 不只依赖目录位置表达关联。

## Ingest

1. 确认来源位于 `raw/inbox/` 或 `raw/library/`。
2. 如需纳入正式知识流，移动或整理到 `raw/library/sources/` 或 `raw/library/notes/`。
3. 更新 `raw-index.md`。
4. 按 Page Shapes 创建或更新 `wiki/sources/`。
5. 基于当前来源主动抽取方法论、概念、问题、决策原则或操作经验，并创建或更新必要的 `wiki/knowledge/`。
6. 结合已有 `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/` 判断可能的综合方向；如已有 synthesis 相关则建立双链，如形成稳定跨来源结论则更新或创建 `wiki/syntheses/`。
7. 在 source、knowledge、synthesis 之间写入 Obsidian `[[...]]` 双链。
8. 更新 `index.md` 和 `log.md`。

移动来源后，不删除空的 inbox bucket 目录。

## Query

1. 优先读取 `wiki/`。
2. 证据不足时回到 `raw/library/` 补查。
3. 输出结论、证据、冲突与缺口。
4. 用户确认写回时，复用上一轮已确认答案，不重新生成不一致版本。
5. 需要写回时，按 Page Shapes 创建或更新 `wiki/knowledge/` 或 `wiki/syntheses/`。

## Lint

1. 检查 `raw/library/` 与 `raw-index.md` 是否一致。
2. 检查 wiki 三桶结构是否清晰。
3. 检查缺失证据、弱连接、孤儿页和 `needs-update` 候选。
4. 默认先报告，不主动大范围修复。

## Naming And Time

- 文件名使用清晰、稳定、可读的短横线命名。
- 中文标题可以保留在页面标题中。
- 时间格式统一为 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。

## Page Shapes

### Frontmatter Rules

所有 wiki 页面必须使用 YAML frontmatter。frontmatter 用于 Obsidian 属性、检索和 Dataview，不替代正文证据与 `[[...]]` 双链。

通用字段：
- `page_type`：`source`、`knowledge` 或 `synthesis`
- `tags`：包含页面类型标签和 3-8 个从内容抽取的关键词
- `created_at`：首次创建时间，格式 `YYYY-MM-DD HH:MM:SS`
- `updated_at`：最近更新时间，格式 `YYYY-MM-DD HH:MM:SS`

类型标签：
- source page 使用 `llm-wiki/source`
- knowledge page 使用 `llm-wiki/knowledge`
- synthesis page 使用 `llm-wiki/synthesis`

source page 额外字段：
- `raw_bucket`：`sources` 或 `notes`，必须与 `raw_path` 所在 bucket 一致

### Source Page

用于 `wiki/sources/`，由 ingest 创建或更新。

必须包含：
- YAML frontmatter：`page_type: source`、`tags`、`created_at`、`updated_at`、`raw_path`、`raw_index_id`、`raw_bucket`
- 基本信息
- 原始路径
- 摘要
- 关键观点
- 证据 / 摘录
- 关联 knowledge，使用 `[[...]]` 双链
- 可能的 syntheses，使用 `[[...]]` 双链
- 待跟进问题

### Knowledge Page

用于 `wiki/knowledge/`，由 ingest 或 query 创建或更新。

必须包含：
- YAML frontmatter：`page_type: knowledge`、`tags`、`created_at`、`updated_at`、`knowledge_type`
- 当前理解
- 关键来源
- 证据
- 相关 knowledge，使用 `[[...]]` 双链
- 不确定或冲突
- 下一步

### Synthesis Page

用于 `wiki/syntheses/`，由 query 或高价值 ingest 创建或更新。

必须包含：
- YAML frontmatter：`page_type: synthesis`、`tags`、`created_at`、`updated_at`、`synthesis_type`
- 范围
- 当前结论
- 共识
- 分歧
- 证据链
- 缺口
- 相关 sources / knowledge，使用 `[[...]]` 双链
```

#### `system/raw-index-rules.md`

```md
# Raw Index Rules

`raw-index.md` 是唯一官方 raw ingest 状态表。它追踪 `raw/library/` 中资料的状态，不替代原文，也不登记尚未纳入 library 的临时 inbox 文件。

## Buckets

- `sources`：文章、书、论文、网页、视频文字稿等外部来源。
- `notes`：用户自己的原始笔记、摘录或临时记录。

raw 下不使用 `syntheses`。

## Status

- `ingested`：已经纳入 wiki 知识流。
- `needs-update`：用户明确要求更新，或存在明确更新信号，必须填写 `update_reason`。

`update_reason` 只属于 `needs-update`。如果来源完成 re-ingest 并回到 `ingested`，不要保留旧的 `update_reason`；re-ingest 原因写入 `log.md`。

## Required Fields

- `id`
- `title`
- `raw_path`
- `status`
- `last_status_at`
- `wiki_page`
- `update_reason`

## Time Format

所有状态时间字段使用 `YYYY-MM-DD HH:MM:SS`，写入前必须先读取当前本地时间。
```

### 6. 最小验证

初始化完成后，至少检查：

- 关键目录是否存在
- 关键根文件是否存在
- `system/workflow.md` 是否存在并包含 `Page Shapes`
- `system/workflow.md` 是否定义 wiki 页面 frontmatter 与 tags 规则
- `system/raw-index-rules.md` 是否存在
- `CLAUDE.md` 是否提到 `raw-index.md`
- `CLAUDE.md` 与 `system/workflow.md` 是否没有继续把 wiki 第二桶定义为 `notes/`
- `raw-index.md` 是否提到 `raw/inbox/`、`raw/library/`、`needs-update`、`update_reason`
- init 是否只生成 schema 允许的 system 文件
- init 是否没有预建 raw/wiki bucket 子目录
- 如果发现历史 `wiki/notes/` 目录，是否只报告而没有自动迁移

### 7. 向用户汇报结果

汇报时要简洁，但要明确说出：

1. 这是全新初始化还是缺失补齐
2. 创建了哪些目录/文件
3. 保留了哪些已有文件
4. 是否发现与当前规则冲突的结构或旧 schema
5. 建议的下一步

推荐汇报结构：

```md
已完成个人 wiki 初始化（全新初始化 / 缺失补齐）。

已创建/补齐：
- ...

已保留：
- ...

注意点：
- ...

下一步建议：
- 导入第一个真实来源时创建 `raw/inbox/sources/`
- 确认后移动到 `raw/library/sources/`
- 在 `raw-index.md` 登记并执行第一次 ingest
```

## 不要这样做

- 不要把 `CLAUDE.md` 放进 `system/` 代替根目录版本
- 不要把 `raw/` 直接做成单层目录而省略 `inbox/` 与 `library/`
- 不要在 raw 下创建 `syntheses/`
- 不要在 init 阶段预建 `raw/inbox/sources/`、`raw/library/sources/`、`wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/`
- 不要创建 schema 之外的规则文件或辅助目录
- 不要创建 `topics/`、`people/`、`concepts/`、`questions/` 目录
- 不要把 `raw-index.md` 变成可有可无的参考文件
- 不要在初始化阶段伪造来源条目
- 不要覆盖用户已有内容而不先检查
- 不要因为已有规则文件存在就静默保留旧 schema
- 不要在 init 中迁移、删除或批量重命名历史 `wiki/notes/` 页面
- 不要把 ingest、query、lint 的职责混到 init 里

## 示例触发

**示例 1**
用户：`帮我给这个目录初始化一个个人 LLM wiki，按最小结构来。`

动作：使用本 skill，检查目录现状，创建或补齐最小骨架，最后汇报结果。

**示例 2**
用户：`这个 Obsidian vault 已经有一点结构了，你帮我补齐到能继续 ingest/query/lint。`

动作：使用本 skill，以缺失补齐模式工作，优先保留已有内容，补齐缺失的根文件、system 文件和基础入口目录；如果已有规则文件仍含旧 schema，只做规则层最小补丁。

**示例 3**
用户：`先别导入资料，先把目录和规则文件搭好。`

动作：使用本 skill，只做初始化，不做 ingest，不创建具体 bucket 子目录或知识页。
