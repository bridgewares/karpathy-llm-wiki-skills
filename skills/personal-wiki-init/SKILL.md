---
name: personal-wiki-init
description: 初始化或补齐个人 LLM Wiki / Obsidian-first 知识库骨架。只要用户是在说新建个人 wiki、初始化 vault、按 llm-wiki 思路搭目录、补齐缺失的 CLAUDE.md/raw-index.md/templates/system 规则文件、或把半成品知识库补成完整结构，就应该主动使用这个 skill，即使用户没有明确说“skill”或“init”。 仅用于骨架初始化与缺失补齐；如果用户是在导入来源、执行 ingest、回答知识问题、检查孤儿页或做 lint，则不要触发这个 skill。
---

# Personal Wiki Init

这个 skill 用于在一个目录中**全新初始化**或**补齐已有但不完整的个人 LLM Wiki**。

目标不是做 ingest、query 或 lint，而是确保 wiki 拥有一套可持续维护的最小骨架，并且与当前项目约定一致。

## 适用场景

当用户表达以下意图时使用：

- “帮我初始化个人 wiki”
- “按这个规范搭一个 llm-wiki”
- “给我的知识库建目录和模板”
- “把这个 vault 补齐到标准结构”
- “检查缺什么并初始化”

如果用户要做的是：

- 导入具体来源 → 这不是本 skill
- 从 wiki 回答问题 → 这不是本 skill
- 检查一致性/孤儿页/needs-update → 这不是本 skill

## 工作模式

本 skill 支持两种模式，但**默认不需要让用户先选模式**，而是先观察当前目录状态后自动决定：

1. **全新初始化**
   - 目录里还没有完整 wiki 骨架。
   - 直接创建目录、根文件、模板和 system 规则文件。

2. **缺失补齐**
   - 已经存在部分结构或部分文件。
   - 只补齐缺失项。
   - 对已存在文件优先保持最小修改，除非用户明确要求统一重写。

## 核心原则

1. **先确认目标 vault 根目录，再动手**
   - 先明确这次要初始化的是哪个目录。
   - 如果用户已经点名目录，就以那个目录为准。
   - 如果用户没有点名目录，默认以当前工作目录作为目标 vault 根目录。

2. **先读现状，再写内容**
   - 先检查当前目录已有的文件与目录。
   - 不要盲目覆盖用户已有内容。

3. **优先补齐，避免重建**
   - 如果文件已经存在且内容明显有用，优先保留。
   - 缺失补齐模式下，默认目标是补洞，而不是统一重写。
   - 只有在用户明确要求或文件内容显著违背当前规则时才重写。

4. **保持中文优先**
   - 文档主体优先中文。
   - 保留必要英文术语，如 ingest、query、lint、source page。

5. **遵守当前架构边界**
   - `raw/inbox/` 是新来源入口。
   - `raw/library/` 是正式原始资料库。
   - `raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。
   - `wiki/` 是知识页层，不与 raw 混用。

6. **初始化只做骨架，不做后续工作**
   - 不导入真实来源。
   - 不创建具体 source page。
   - 不虚构 raw-index 条目。
   - 不自动创建大量空知识页。

## 标准目标结构

初始化完成后，目标结构应至少包括：

```text
CLAUDE.md
raw-index.md
index.md
log.md
inbox.md
raw/
  inbox/
    articles/
    videos/
    books/
    podcasts/
    webclips/
    pdfs/
    images/
  library/
    articles/
    videos/
    books/
    podcasts/
    webclips/
    pdfs/
    images/
wiki/
  sources/
    articles/
    videos/
    books/
    podcasts/
    webclips/
  topics/
  people/
  concepts/
  projects/
  questions/
  syntheses/
  timelines/
templates/
  source-note.md
  topic-page.md
  person-page.md
  concept-page.md
  question-page.md
  synthesis-page.md
system/
  workflow.md
  taxonomy.md
  naming-conventions.md
  raw-index-rules.md
```

## 初始化流程

按下面顺序执行。

### 1. 识别当前状态

先检查：

- 当前工作目录是否就是目标 vault 根目录
- 如果用户明确给了路径，是否应以该路径作为目标 vault 根目录
- 根目录是否已有 `CLAUDE.md`
- 是否已有 `raw/`、`wiki/`、`templates/`、`system/`
- 是否已有 `raw-index.md`、`index.md`、`log.md`、`inbox.md`
- `raw/` 是否已经是 `inbox + library` 双层结构
- `system/` 是否已有支持规则文件
- `templates/` 是否已有核心模板

然后判断：

- 几乎都不存在 → 视为**全新初始化**
- 已有部分结构 → 视为**缺失补齐**

### 2. 创建或补齐目录

需要确保以下目录存在：

- `raw/inbox/articles/`
- `raw/inbox/videos/`
- `raw/inbox/books/`
- `raw/inbox/podcasts/`
- `raw/inbox/webclips/`
- `raw/inbox/pdfs/`
- `raw/inbox/images/`
- `raw/library/articles/`
- `raw/library/videos/`
- `raw/library/books/`
- `raw/library/podcasts/`
- `raw/library/webclips/`
- `raw/library/pdfs/`
- `raw/library/images/`
- `wiki/sources/articles/`
- `wiki/sources/videos/`
- `wiki/sources/books/`
- `wiki/sources/podcasts/`
- `wiki/sources/webclips/`
- `wiki/topics/`
- `wiki/people/`
- `wiki/concepts/`
- `wiki/projects/`
- `wiki/questions/`
- `wiki/syntheses/`
- `wiki/timelines/`
- `templates/`
- `system/`

### 3. 创建或补齐根文件

确保以下文件存在并符合当前规则：

- `CLAUDE.md`
- `raw-index.md`
- `index.md`
- `log.md`
- `inbox.md`

#### `CLAUDE.md` 要点

必须清楚定义：

- 这是个人 LLM 维护的 wiki
- `raw/` 与 `wiki/` 的职责边界
- `raw/inbox/` 与 `raw/library/` 的区别
- `raw-index.md` 是唯一官方 ingest 状态表
- ingest/query 工作流
- 页面创建与更新原则
- `index.md` 与 `log.md` 的维护要求
- 中文优先与秒级时间格式要求

#### `raw-index.md` 要点

必须清楚定义：

- 它是唯一官方 raw ingest 状态表
- 当前阶段暂不使用单文件 metadata
- 只有 `raw/library/` 中的来源才登记
- 状态至少包含：`new`、`ingested`、`needs-update`
- `needs-update` 必须带 `update_reason`
- 所有状态时间字段使用 `YYYY-MM-DD HH:MM:SS`
- 可以提供按类型分区的空白列表骨架

#### `index.md` 要点

必须包含：

- 当前重点主题入口
- Raw 管理入口
- 来源页目录入口
- 知识页目录入口
- 最近新增入口

#### `log.md` 要点

初始化后至少要有一条初始化记录，说明：

- 已创建基础结构
- 已初始化 `raw-index.md`
- 已初始化根目录规则与模板
- 下一步建议是导入第一个真实来源并测试 ingest

#### `inbox.md` 要点

至少包含：

- 待处理来源
- 待整理问题
- 待执行动作

### 4. 创建或补齐模板文件

确保以下模板存在：

- `templates/source-note.md`
- `templates/topic-page.md`
- `templates/person-page.md`
- `templates/concept-page.md`
- `templates/question-page.md`
- `templates/synthesis-page.md`

模板应保持：

- 结构化
- 最小可用
- 中文优先
- 不预填虚构内容

### 5. 创建或补齐 system 规则文件

确保以下文件存在：

- `system/workflow.md`
- `system/taxonomy.md`
- `system/naming-conventions.md`
- `system/raw-index-rules.md`

这些文件应分别覆盖：

- ingest/query/lint 工作流
- 页面分类法
- 命名规范与时间格式
- raw-index 维护规则与状态语义

### 6. 做一次最小验证

初始化完成后，至少检查：

- 关键目录是否存在
- 关键根文件是否存在
- 模板文件是否存在
- system 文件是否存在
- `CLAUDE.md` 是否提到 `raw-index.md`
- `raw-index.md` 是否提到 `raw/inbox/`、`raw/library/`、`needs-update`、`update_reason`

### 7. 向用户汇报结果

汇报时要简洁，但要明确说出：

1. 这是全新初始化还是缺失补齐
2. 创建了哪些目录/文件
3. 保留了哪些已有文件
4. 是否发现与当前规则冲突的旧结构
5. 建议的下一步

## 输出风格

默认用简洁中文汇报，避免长篇解释。

推荐汇报结构：

```md
已完成个人 wiki 初始化（全新初始化 / 缺失补齐）。

已创建/补齐：
- ...
- ...

已保留：
- ...

发现的旧结构或注意点：
- ...

下一步建议：
- 导入第一个真实来源到 `raw/inbox/<type>/`
- 确认后移动到 `raw/library/<type>/`
- 在 `raw-index.md` 登记并执行第一次 ingest
```

## 不要这样做

- 不要把 `CLAUDE.md` 放进 `system/` 代替根目录版本
- 不要把 `raw/` 直接做成单层类型目录而省略 `inbox/` 与 `library/`
- 不要把 `raw-index.md` 变成可有可无的参考文件
- 不要在初始化阶段伪造来源条目
- 不要一次性生成大量空 topic/person/concept 页面
- 不要覆盖用户已有内容而不先检查
- 不要把 ingest、query、lint 的职责混到 init 里

## 示例触发

**示例 1**
用户：`帮我给这个目录初始化一个个人 LLM wiki，按我们之前定的规范来。`

动作：使用本 skill，检查目录现状，创建或补齐骨架，最后汇报结果。

**示例 2**
用户：`这个 Obsidian vault 已经有一点结构了，你帮我补齐成完整的 wiki 规范。`

动作：使用本 skill，以缺失补齐模式工作，优先保留已有内容。

**示例 3**
用户：`基于 llm-wiki 的思路，先别导入资料，先把目录、模板和规则文件都搭好。`

动作：使用本 skill，只做初始化，不做 ingest。
