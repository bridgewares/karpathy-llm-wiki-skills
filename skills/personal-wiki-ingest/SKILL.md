---
name: personal-wiki-ingest
description: 当用户要把一份或一批“来源材料”纳入个人 wiki 的正式来源库时，使用这个 skill：无论是首次入库、批量处理待处理材料，还是对已入库来源进行重新 ingest / 重建 / 刷新。尤其适用于用户提到把文档从 inbox 处理进 library、按文件或路径处理、登记或更新 raw-index、因原文变化或结构变化而重做、保留原 source identity、避免重复来源页，并同步相关 source/topic/concept/person/question 页面。 把“处理这些文档”“整理进库”“刷新旧来源”“同步相关页面”这类以来源为中心的请求，也视为应触发。 不要用于：初始化或补齐 wiki 骨架；纯问答、总结、synthesis；lint/巡检；或不围绕具体来源入库的单个 topic/concept/person/question 页面编辑。
---

# Personal Wiki Ingest

这个 skill 用于把一个具体来源从 `raw/inbox/` 推进到 wiki 知识层。

目标不是只做目录整理，而是完成一次完整 ingest：

- 确认来源类型
- 整理到 `raw/library/`
- 维护 `raw-index.md`
- 创建或更新 `wiki/sources/...` 来源页
- 提取并写入结构化属性（不只标签）
- 自动更新相关 `topic / concept / person / question` 页面
- 更新 `index.md`
- 追加 `log.md`

## 适用场景

当用户表达以下意图时使用：

- “把这篇文章 ingest 到 wiki”
- “处理这个 PDF / transcript / webclip”
- “把 raw/inbox 里的这个来源整理进知识库”
- “给这个来源建 source page，并更新相关页面”
- “重新 ingest 这个已有来源”
- “这个来源变了，帮我同步到 wiki”

如果用户要做的是：

- 初始化骨架 → 这不是本 skill
- 单纯回答 wiki 里的问题 → 这不是本 skill
- 检查孤儿页、交叉引用、needs-update → 这不是本 skill
- 只新建某个 topic / concept / person 页面而没有具体来源 → 这不是本 skill

## 工作模式

本 skill 默认支持三类 ingest：

1. **首次 ingest 新来源**
   - 来源当前在 `raw/inbox/<type>/`
   - 或刚整理进入 `raw/library/<type>/`
   - 需要首次登记 `raw-index.md`
   - 需要创建对应来源页并同步相关知识页

2. **重新 ingest / update 既有来源**
   - 来源已在 `raw/library/<type>/`
   - `raw-index.md` 中已有条目
   - 由于 raw 内容变化、交叉引用补齐、模板变化或人工要求，需要重做
   - 需要把相关 wiki 页面同步到最新状态

3. **批量 ingest 多个待处理来源**
   - 用户没有指定单一来源
   - 当前 vault 中存在多个待 ingest 候选来源
   - 应按候选来源逐个处理，而不是只挑一个来源停下
   - 处理后每个来源都要完成状态登记、来源页创建/更新和相关知识页同步

## 核心原则

1. **先确认目标来源，再执行 ingest**
   - 先明确这次处理的是哪个来源文件或来源对象。
   - 如果用户点名了路径，就以该路径为准。
   - 如果用户没有点名，但只给了一个候选来源，可以先用它。
   - 如果存在多个候选来源且无法安全判断，先澄清，不要盲猜。

2. **先判断来源类型，再落位**
   - 来源必须归到明确类型：`articles`、`videos`、`books`、`podcasts`、`webclips`、`pdfs`、`images`。
   - 新来源先进入 `raw/inbox/<type>/`。
   - 确认后整理到 `raw/library/<type>/`。

3. **`raw-index.md` 是唯一官方状态表**
   - 当前阶段暂不使用单文件 metadata 代替它。
   - 只有进入 `raw/library/` 的来源才登记。
   - 状态字段必须遵守秒级时间格式：`YYYY-MM-DD HH:MM:SS`。

4. **默认自动更新相关知识页**
   - ingest 完成来源页后，不要停在 `wiki/sources/...`。
   - 只要来源里有足够信息，就默认继续更新相关 `topic / concept / person / question` 页面。
   - 这样做是为了保持知识库更准实时，减少后续补写造成的污染和偏移。

5. **先提取结构化属性，再做跨页同步**
   - ingest 不应停留在“移动文件 + 建链接”。
   - 除标签外，还应尽量提取：摘要、关键观点、人物、概念、主题、问题、证据/摘录等结构化属性。
   - 这些属性主要写入 `wiki/sources/...`，作为后续跨页同步的稳定中间层。
   - 这样做是为了减少知识散落、降低后续人工补写成本，并让相同标签或相近主题的来源更容易被关联起来。

6. **自动更新不等于无限扩张**
   - 自动更新相关页面，但仍要克制。
   - 优先更新已有页面。
   - 只有长期值得追踪的主题、人物、概念、问题才新建独立页面。
   - 对信息不足的实体，先留在来源页，不强行拆页。

7. **区分首次 ingest 与重新 ingest**
   - 首次 ingest：来源进入库、登记、建来源页、同步相关页、记日志。
   - 重新 ingest：保留来源身份，更新状态字段、来源页和相关知识页，并说明 `update_reason`。

8. **不把猜测写成事实**
   - 来源页、topic 页、concept 页、person 页、question 页都要区分：结论 / 证据 / 未解决问题。
   - 发现冲突信息要标出冲突，不要强行合并成单一结论。

## ingest 目标结果

一次成功 ingest 之后，通常应当得到这些结果：

- 来源位于 `raw/library/<type>/`
- `raw-index.md` 中存在对应条目
- 条目状态正确：`new` / `ingested` / `needs-update`
- 如为 `needs-update`，必须带 `update_reason`
- `wiki/sources/...` 中存在对应来源页
- 相关 `topic / concept / person / question` 页面已自动同步
- `index.md` 已更新高价值导航
- `log.md` 已追加 ingest 记录

## 执行流程

按下面顺序执行。

### 1. 确认来源与当前状态

先检查：

- 目标来源路径是什么
- 来源当前位于 `raw/inbox/` 还是 `raw/library/`
- 来源属于哪种类型
- `raw-index.md` 中是否已有该来源条目
- 是否已有对应 `wiki/sources/...` 来源页
- 是否存在 `needs-update` 或重新 ingest 信号

重新 ingest 的常见信号：

- raw 内容发生实质变化
- 模板变化
- 分类法变化
- 新交叉引用需要补回旧来源
- 用户明确要求重做

### 2. 整理来源到 `raw/library/`

如果来源还在 `raw/inbox/<type>/`：

- 先确认类型正确
- 再整理或移动到 `raw/library/<type>/`

如果来源已经在 `raw/library/<type>/`：

- 不重复制造副本
- 直接进入状态检查与更新流程

### 3. 建立或更新 `raw-index.md` 条目

必须保证条目至少覆盖这些信息：

- 标题
- `raw_path`
- `source_type`
- `status`
- `source_page`
- `registered_at`
- `raw_updated_at`
- `last_ingested_at`
- `update_reason`
- `notes`

状态规则：

- `new`：已入 `raw/library/`，已登记，但尚未完成第一次 ingest
- `ingested`：已完成 ingest，已有来源页，且当前无更新触发
- `needs-update`：需要重做，且必须带 `update_reason`

建议的 `update_reason`：

- `source-changed`
- `wiki-structure-changed`
- `template-changed`
- `crosslink-needed`
- `manual-review`

### 4. 创建或更新 `wiki/sources/...` 来源页

来源页是 ingest 的主承载页，优先写结构化属性，再据此同步到其他知识页。

来源页至少应包含：

- 来源类型
- 原始路径
- 作者 / 主讲人
- 日期
- 状态
- 标签
- 一句话摘要
- 核心内容
- 关键观点
- 重要摘录
- 相关人物
- 相关概念
- 相关主题
- 待跟进问题

如果来源信息足够，优先补这些结构化属性：

- `tags`：便于把同类来源串起来
- `summary`：压缩来源主旨
- `key_points`：来源里的关键论点或发现
- `people`：值得跟踪的人物
- `concepts`：稳定概念或术语
- `topics`：跨来源主题
- `questions`：来源回答了什么，或提出了什么未解问题
- `evidence` / `key_quotes`：支撑结论的原句、摘录、事实点

这些属性默认应写进 `wiki/sources/...`，而不是把 raw 原文改造成主 metadata 库。
`raw-index.md` 继续只承担来源状态登记职责；必要时可在 `notes` 中写一句简短提示，但不要把完整知识提取塞进去。

来源页的任务是回答：
- 这个来源说了什么
- 哪些观点值得后续同步到知识页
- 哪些实体需要链接或沉淀

### 5. 自动更新相关知识页

这是本 skill 的默认动作，不需要用户额外要求。

#### Topic 页面

- 如果相关 `wiki/topics/...` 页面已存在，优先更新它
- 如果来源明确补充了长期主题，允许新建最小 topic 页面
- 更新：当前理解、关键来源、子主题、未解决问题

#### Concept 页面

- 如果来源提供了稳定可定义的概念，更新或新建 concept 页面
- 更新：定义、直观解释、为什么重要、例子、相关来源、常见误解

#### Person 页面

- 如果来源涉及值得长期追踪的人物，更新或新建 person 页面
- 更新：人物简介、为什么重要、核心观点、相关作品与来源、关联主题

#### Question 页面

- 如果来源回答了已有问题，更新对应 question 页面
- 如果来源暴露出新的长期问题，允许新建最小问题页
- 更新：当前回答、支持证据、冲突证据、仍缺少什么、下一步资料

### 6. 更新 `index.md`

- 只更新高价值导航，不追求穷举全部文件
- 如果来源或相关页面值得出现在导航中，就补进去
- 避免把 `index.md` 变成文件清单

### 7. 追加 `log.md`

每次 ingest 都要记录。

记录至少说明：

- 日期
- 操作类型：ingest 或 re-ingest
- 处理了哪个来源
- 更新了哪些页面
- 是否发生跨页同步
- 如果是重新 ingest，原因是什么

### 8. 最小验证

完成后至少检查：

- 来源是否已在 `raw/library/<type>/`
- `raw-index.md` 是否有对应条目
- 条目状态与时间字段是否合理
- 来源页是否存在并已更新
- 相关 `topic / concept / person / question` 页面是否已同步
- `index.md` 是否已更新
- `log.md` 是否已追加记录

## 输出风格

默认用简洁中文汇报，避免长篇解释。

推荐汇报结构：

```md
已完成来源 ingest（首次 ingest / 重新 ingest）。

处理来源：
- ...

已更新：
- raw/library/... 
- raw-index.md
- wiki/sources/...
- wiki/topics/...
- wiki/concepts/...
- wiki/people/...
- wiki/questions/...
- index.md
- log.md

当前状态：
- status: ingested / needs-update
- update_reason: ...

注意点：
- ...

下一步建议：
- ...
```

## 不要这样做

- 不要跳过 `raw-index.md`，也不要把它降级成可选参考文件
- 不要只建来源页而不更新相关知识页
- 不要把 `needs-update` 裸用而不写 `update_reason`
- 不要在未确认来源类型时随意归类
- 不要为信息不足的实体批量制造碎片页面
- 不要把 query / lint 的职责混进 ingest
- 不要把猜测写成来源明确说过的话

## 示例触发

**示例 1**
用户：`把 raw/inbox/articles 里的这篇文章 ingest 到 wiki，并同步相关 topic 页面。`

动作：使用本 skill，完成 raw 整理、状态登记、来源页创建和相关页面同步。

**示例 2**
用户：`这个 PDF 之前处理过，但我刚更新了原文，请重新 ingest，并把相关 concept 页面一起更新。`

动作：使用本 skill，以重新 ingest 模式工作，更新 `raw-index.md` 状态、来源页和相关知识页。

**示例 3**
用户：`帮我把这个 webclip 从 inbox 整理进 library，建 source page，并自动把涉及的人物和问题页补上。`

动作：使用本 skill，执行完整 ingest，并自动更新相关 person / question 页面。
