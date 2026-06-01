---
name: personal-wiki-query
description: 基于个人 LLM Wiki 已有内容进行查询、回答、比较、综合、核对、写文章或将高价值答案写回 wiki 时使用。适用于用户询问 wiki 当前怎么说、要求比较来源或知识页、要求基于 wiki 写一篇文章、或确认把上一轮回答沉淀到 wiki。不要用于新来源 ingest、移动 raw 文件、维护 raw-index、初始化骨架或 lint 巡检。
---

# Personal Wiki Query

这个 skill 用于基于已有个人 wiki 回答问题、比较材料、综合结论、写文章，并在合适时把结果沉淀回 `wiki/knowledge/` 或 `wiki/syntheses/`。

query 不导入新来源，不移动 raw 文件，不维护 `raw-index.md`。它可以读取 `raw/library/` 补证据，但不能把补证据过程变成 ingest。

本 skill 以 `personal-wiki-init` 和 `personal-wiki-ingest` 的当前 schema 为准：

- raw 只使用 `sources/`、`notes/`
- wiki 只使用 `sources/`、`knowledge/`、`syntheses/`
- `wiki/sources/` 是所有 raw item 的来源追踪层，不是 raw `sources/` bucket 的镜像
- `wiki/knowledge/` 是结构化知识页，不存在 `wiki/notes/`
- `wiki/syntheses/` 是跨来源、跨知识页的综合页
- 页面结构以 `system/workflow.md` 的 `Page Shapes` 为准
- wiki 页面必须使用 Obsidian `[[...]]` 双链和 YAML frontmatter

## 适用场景

当用户表达以下意图时使用：

- “这个 wiki 里现在怎么回答这个问题？”
- “帮我看看这个主题当前有哪些结论和证据”
- “比较这几页 / 这几份来源的观点”
- “总结一下这个问题目前在知识库里的答案”
- “先基于现有 wiki 回答，不够再回原文看”
- “基于这些内容写一篇文章”
- “把上面的回答写入 md / 写回 wiki”
- “这个结论值得沉淀吗，帮我整理进去”

如果用户要做的是：

- 初始化骨架 -> 这不是本 skill
- 导入具体 raw 来源、移动文件、登记 `raw-index.md`、创建来源追踪页 -> 这不是本 skill
- 巡检孤儿页、交叉引用、一致性、`needs-update` -> 这不是本 skill
- 只改写一个现有页面的文字表达，且没有查询、比较、综合或写作意图 -> 这不是本 skill

## 工作模式

1. **问答**
   - 基于现有 wiki 回答用户问题。
   - 输出当前结论、依据页面、证据、冲突和缺口。

2. **比较 / 综合**
   - 比较多个 `wiki/sources/`、`wiki/knowledge/` 或 `wiki/syntheses/` 页面。
   - 形成共识、分歧、证据链和阶段性结论。

3. **文章写作**
   - 当用户明确要求“写一篇文章”“整理成文章”“生成 md 文稿”时，可以直接写作。
   - 文章可以基于 wiki 与必要 raw 证据生成，不需要先询问是否允许写。
   - 如果用户要求保存为文件，按用户指定路径写；未指定路径时，先给正文并询问落点。

4. **写回沉淀**
   - 用户明确要求写回 wiki 时，直接执行。
   - 如果 query 结果明显适合长期复用，可以主动建议写入 `wiki/knowledge/` 或 `wiki/syntheses/`。
   - 用户回答“写入”“记下来”“可以”时，必须以上一轮已确认回答为底稿，不重新生成另一版结论。

## 核心原则

1. **优先读 wiki**
   - 先读 `index.md` 判断入口。
   - 优先读取相关 `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/`。
   - 不要在 wiki 足够时直接跳到 raw。

2. **必要时才补 raw 证据**
   - 当 wiki 证据不足、页面冲突、结论过强或用户明确要求核对原文时，才读取 `raw/library/`。
   - 读取 raw 只为当前 query 补证据，不移动文件，不登记状态，不创建 raw-index 条目。

3. **source / knowledge / synthesis 边界清楚**
   - `wiki/sources/`：来源追踪页，回答“这个 raw item 说了什么”。
   - `wiki/knowledge/`：可复用的方法论、概念、问题、决策原则、操作经验或判断框架。
   - `wiki/syntheses/`：跨多个 sources / knowledge pages 的比较、共识、分歧和阶段性结论。
   - query 写回只写 `wiki/knowledge/` 或 `wiki/syntheses/`；不要创建 `wiki/notes/`。

4. **默认回答，不默认落盘**
   - 普通查询先给答案。
   - 高价值答案可以建议沉淀，但不要在用户未明确同意时静默写文件。
   - 用户明确要求写文章时，可以直接生成文章内容。

5. **确认写回时复用上一轮答案**
   - 如果用户在上一轮 query 后说“写入 md / 写回 wiki / 记下来”，必须复用上一轮已确认答案作为底稿。
   - 写回时可以结构化、补 frontmatter、补 `[[...]]`、去重和调整小节，但不能重新生成一个结论不同的版本。
   - 如果当前上下文无法可靠取得上一轮答案，必须说明无法保证一致，并让用户提供要写回的文本或确认重新生成。

6. **不把猜测写成事实**
   - 明确区分结论、证据、摘录、推断、不确定和冲突。
   - 没有证据支撑的内容只能写为“不确定”“待查证”或“下一步”。

## 执行流程

### 1. 判断用户意图

先判断本次属于：

- 回答问题
- 比较 / 综合
- 写文章
- 写回沉淀

如果用户同时要求回答和写文章，先完成文章；如果文章需要 wiki 证据，先读 wiki 再写。

如果用户是基于上一轮回答要求写回，直接进入“写回沉淀”，不要重新查询生成不同内容。

### 2. 读取导航与相关页面

优先读取：

- `index.md`
- 相关 `wiki/sources/...`
- 相关 `wiki/knowledge/...`
- 相关 `wiki/syntheses/...`

读取重点：

- 当前结论
- 关键来源
- 证据 / 摘录
- 已有 `[[...]]` 链接
- 冲突、不确定和待跟进问题
- 页面 frontmatter 的 `tags`、`page_type`、`updated_at`

### 3. 必要时补查 raw

只有以下情况才读取 `raw/library/`：

- wiki 证据不足
- wiki 页面之间有冲突，需要回原文核对
- 用户明确要求看原文
- 写文章需要更准确的摘录或事实支撑

补查后仍要把结论落回 wiki 证据链，不要让回答只依赖散落 raw 摘录。

### 4. 生成回答或文章

普通回答默认包含：

- 结论
- 依据页面 / 来源
- 冲突或不确定
- 下一步

比较 / 综合默认包含：

- 范围
- 共识
- 分歧
- 证据链
- 缺口
- 是否建议写回 synthesis

文章写作默认要求：

- 有清晰标题
- 结构完整，适合直接保存为 md
- 关键判断能追溯到 wiki 页面或 raw 证据
- 不把证据不足的推断写成确定事实
- 如用户要求特定风格、受众或长度，以用户要求为准

### 5. 判断是否建议沉淀

满足以下情况时，主动建议写回：

- 这是会反复被问到的稳定问题
- 形成了可复用方法论、概念、原则、操作经验或判断框架
- 比较了多个 sources / knowledge pages，并形成阶段性结论
- 修正了已有 wiki 的缺口、冲突或过强结论
- 文章本身适合作为长期 synthesis 或 knowledge 页面沉淀

建议落点：

- `wiki/knowledge/`：单主题知识、方法论、概念、问题、原则、实践经验。
- `wiki/syntheses/`：跨来源比较、专题文章、阶段性综合、共识与分歧。

如果只是一次性回答、临时解释或证据不足，不建议写回。

### 6. 写回 `wiki/knowledge/`

写回 knowledge page 时：

- 需要目录时再创建 `wiki/knowledge/`
- 优先更新已有相关页面
- 只有没有合适页面时才新建
- 遵守 `system/workflow.md` 的 `Knowledge Page`
- frontmatter 至少包含：
  - `page_type: knowledge`
  - `tags`
  - `created_at`
  - `updated_at`
  - `knowledge_type`
- 正文至少包含：
  - 当前理解
  - 关键来源，使用 `[[...]]` 链接到 source page
  - 证据
  - 相关 knowledge，使用 `[[...]]`
  - 不确定或冲突
  - 下一步

`knowledge_type` 可使用：`methodology`、`concept`、`question`、`principle`、`practice`、`framework`。如果不确定，选择最贴近页面主旨的一个。

### 7. 写回 `wiki/syntheses/`

写回 synthesis page 时：

- 需要目录时再创建 `wiki/syntheses/`
- 优先更新已有相关综合页
- 只有形成稳定跨来源结论时才新建
- 遵守 `system/workflow.md` 的 `Synthesis Page`
- frontmatter 至少包含：
  - `page_type: synthesis`
  - `tags`
  - `created_at`
  - `updated_at`
  - `synthesis_type`
- 正文至少包含：
  - 范围
  - 当前结论
  - 共识
  - 分歧
  - 证据链
  - 缺口
  - 相关 sources / knowledge，使用 `[[...]]`

`synthesis_type` 可使用：`consensus`、`conflict`、`comparison`、`decision`、`research-thread`、`article`。如果用户明确要求写一篇文章并沉淀，通常使用 `article` 或 `research-thread`。

### 8. 写普通 md 文件

如果用户明确要求“写一篇文章”或“保存成 md 文件”：

- 用户指定路径时，直接写到该路径。
- 用户未指定路径时，先给出正文，并建议一个 `wiki/syntheses/...` 或用户指定目录的落点。
- 如果这是上一轮回答的写入请求，必须复用上一轮答案，不重新生成。
- 普通 md 文件不一定是 wiki page；若写入 wiki，则必须遵守 Page Shapes。

### 9. 更新 index.md

只有实际写入或更新 md 文件时才维护 `index.md`。纯回答不更新 `index.md`。

写入后必须同步优化 `index.md`：

- 新增或更新的重要 `wiki/knowledge/` 页面，加入高价值入口。
- 新增或更新的重要 `wiki/syntheses/` 页面，加入高价值入口。
- 用户指定路径保存的文章，如果属于长期可复用内容，加入合适入口；一次性草稿不强行加入。
- 如果本次写入解决了 `index.md` “待处理”里的事项，应从“待处理”移除。
- 在“最近新增”追加一条简短记录，说明时间、动作类型、更新页面和是否复用上一轮答案。

维护规则：

- “待处理”只保留尚未完成、需要后续 action 的事项。
- 已完成的写回、文章保存、synthesis 更新，不要继续放在“待处理”。
- “最近新增”使用真实当前本地时间，格式 `YYYY-MM-DD HH:MM:SS`，写入前必须读取时间。
- 不要把 `log.md` 的完整记录复制到 `index.md`；`index.md` 只保留高价值摘要入口。

### 10. 更新 log.md

只有实际写文件或更新 wiki 页面时才追加 `log.md`。

记录至少包含：

- 当前本地时间，格式 `YYYY-MM-DD HH:MM:SS`，写入前必须读取真实时间
- 操作类型：`query-writeback`、`article-write` 或 `synthesis-update`
- 更新路径
- 依据页面 / 来源
- 是否复用了上一轮答案

纯回答不默认写日志。

## 最小验证

完成后检查：

- 是否没有移动 raw 文件、没有维护 `raw-index.md`
- 是否优先读取了 `wiki/`
- 如果读取 raw，是否确实是为当前 query 补证据
- 回答或文章是否区分结论、证据、冲突和缺口
- 是否没有创建 `wiki/notes/`
- 写回 knowledge/synthesis 时，frontmatter 是否符合 Page Shapes
- 写回页面是否包含必要 `[[...]]` 双链
- 如果用户是二次确认写回，内容是否以上一轮回答为底稿
- 如果写文件，`index.md` 是否更新高价值入口、清理已完成待处理项并追加“最近新增”
- 如果写文件，`log.md` 是否追加记录

## 汇报格式

普通回答：

```md
结论：
- ...

依据页面 / 来源：
- ...

冲突或不确定：
- ...

下一步：
- ...

沉淀建议：
- 可写回 / 暂不建议写回，因为 ...
```

已写回：

```md
已写回：
- wiki/knowledge/... 或 wiki/syntheses/... 或 用户指定路径

已更新：
- index.md
- log.md

依据：
- ...

注意点：
- 是否复用上一轮答案
- 是否仍有证据缺口
```

## 不要这样做

- 不要把 query 做成 ingest：不要移动文件、不要登记 `raw-index.md`、不要创建 raw 来源入库状态。
- 不要创建 `wiki/notes/`。
- 不要在 wiki 已足够时直接跳到 raw。
- 不要把一次轻量回答默认写入 wiki。
- 不要在用户确认“把上面写进去”时重新生成不同结论。
- 不要把普通文章写作误判为必须先问是否写；用户明确要写文章时可以直接写。
- 不要把 source page 当作 knowledge page；source page 是证据追踪，knowledge page 是结构化知识。
- 不要把缺证据推断写成已确认事实。

## 示例触发

**示例 1**
用户：`这个 wiki 里现在怎么回答“如何减少知识漂移”？`

动作：使用本 skill，读取 `index.md`、相关 `wiki/knowledge/` 和 `wiki/sources/`，给出结论、依据、冲突和缺口。

**示例 2**
用户：`比较一下这几份来源关于 agent memory 的共识和分歧。`

动作：使用本 skill，比较相关 `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/`，必要时建议写回 synthesis。

**示例 3**
用户：`基于这些材料写一篇文章。`

动作：使用本 skill，直接写文章；如果需要保存但用户没给路径，先给正文并建议落点。

**示例 4**
用户：`把上面的回答写入 wiki。`

动作：使用本 skill，复用上一轮已确认答案，按 Page Shapes 写入 `wiki/knowledge/` 或 `wiki/syntheses/`，更新 `index.md` 与 `log.md`，不重新生成不同版本。
