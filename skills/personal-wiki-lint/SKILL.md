---
name: personal-wiki-lint
description: 对个人 LLM Wiki 做巡检、健康检查、一致性审计、缺失文档检查、冲突文档识别、弱连接排查、frontmatter 校验或 needs-update 候选判断时使用。适用于检查 raw/library、raw-index、wiki/sources、wiki/knowledge、wiki/syntheses 之间是否一致。不要用于初始化、ingest 新来源、query 问答、写文章或普通单页编辑。
---

# Personal Wiki Lint

这个 skill 用于审计个人 wiki 的结构、状态、链接、证据和内容一致性。

lint 默认只读和汇报，不默认修复。只有用户明确说“修复”“补上”“更新状态”“写回结果”“记日志”时，才执行写操作。

本 skill 以 `personal-wiki-init`、`personal-wiki-ingest`、`personal-wiki-query` 的当前 schema 为准：

- raw 只使用 `sources/`、`notes/`
- wiki 只使用 `sources/`、`knowledge/`、`syntheses/`
- `wiki/sources/` 是所有 raw item 的来源追踪层，不是 raw `sources/` bucket 的镜像
- `wiki/knowledge/` 是结构化知识页，不存在 `wiki/notes/`
- `wiki/syntheses/` 是跨来源、跨知识页的综合页
- `raw-index.md` 是唯一官方 raw ingest 状态表
- 页面结构以 `system/workflow.md` 的 `Page Shapes` 为准

## 适用场景

当用户表达以下意图时使用：

- “帮我做一次 wiki 巡检”
- “检查 raw-index 和 raw/library 是否一致”
- “看看哪些 raw 文件没有 wiki source page”
- “找缺失的 knowledge / synthesis 文档”
- “找冲突文档 / 冲突说法”
- “检查孤儿页、弱连接页、缺失交叉引用”
- “检查 frontmatter、tags、page_type、raw_bucket”
- “哪些来源应该标 needs-update？”
- “帮我列一个后续维护清单”

如果用户要做的是：

- 初始化骨架 -> 这不是本 skill
- 导入新 raw 来源、移动文件、登记 raw-index、创建来源追踪页 -> 这不是本 skill
- 围绕某个问题回答、比较、综合或写文章 -> 这不是本 skill
- 只改写单个页面表达，且没有巡检意图 -> 这不是本 skill

## 巡检范围

默认检查：

- `index.md`
- `raw-index.md`
- `log.md`
- `system/workflow.md`
- `system/raw-index-rules.md`
- `raw/library/sources/`
- `raw/library/notes/`
- `wiki/sources/`
- `wiki/knowledge/`
- `wiki/syntheses/`

如果某个按需目录不存在，不直接判错；先判断当前 vault 是否已经发生过需要该目录的行为。

## 核心原则

1. **先审计，后修复**
   - 默认只输出发现、证据、影响和建议动作。
   - 不静默创建页面、补链接、改状态或写日志。
   - 用户明确要求修复时，才做最小写操作。

2. **缺失文档必须陈述清楚**
   - 发现 raw item 缺少 `wiki/sources/` 追踪页时，列出 raw 路径。
   - 发现 source page 暗示了稳定知识但缺少 `wiki/knowledge/` 页时，列出 source page 与候选 knowledge 主题。
   - 发现多个页面形成综合方向但缺少 `wiki/syntheses/` 页时，列出相关页面和建议综合主题。
   - 不要只说“有缺失”，必须指出缺失什么、依据是什么、建议走 ingest 还是 query 写回。

3. **冲突文档必须陈述清楚**
   - 发现页面之间有冲突说法时，列出冲突双方或多方页面。
   - 说明冲突点、各自证据、可能影响。
   - 区分真正来源冲突、下游总结过强、证据不足和命名不一致。

4. **raw-index 是状态中心**
   - 只有 `raw/library/` 中的资料登记到 `raw-index.md`。
   - `raw/inbox/` 中的文件不应登记。
   - 状态只允许 `ingested`、`needs-update`。
   - `needs-update` 必须有 `update_reason`。
   - `ingested` 不应保留 `update_reason`。

5. **wiki 三桶职责不可混淆**
   - `wiki/sources/` 记录 raw item 说了什么。
   - `wiki/knowledge/` 承载可复用知识。
   - `wiki/syntheses/` 承载跨来源综合。
   - 不创建、不建议创建 `wiki/notes/`。

6. **frontmatter 与双链都要检查**
   - frontmatter 用于 Obsidian 属性、检索和 Dataview。
   - `[[...]]` 双链用于表达知识关系。
   - tags 不能替代双链，目录位置也不能替代双链。

7. **不把猜测写成事实**
   - 证据不足时写“可能”“建议核对”。
   - 不把可疑信号直接升级为已确认冲突或已确认陈旧。

## 执行流程

### 1. 判断任务边界

确认用户是在做巡检、健康检查、一致性审计或维护清单。

如果用户实际要求 ingest、query、init 或单页编辑，转用对应 skill，不要用 lint 偷偷完成。

### 2. 建立当前结构视图

读取并记录：

- 是否存在 `CLAUDE.md`、`raw-index.md`、`index.md`、`log.md`
- 是否存在 `system/workflow.md` 和 `system/raw-index-rules.md`
- `raw/library/sources/` 和 `raw/library/notes/` 当前文件
- `wiki/sources/`、`wiki/knowledge/`、`wiki/syntheses/` 当前页面
- `index.md` 是否指向重要页面
- `index.md` 的“待处理”是否只包含尚未完成的事项
- `index.md` 的“最近新增”是否覆盖最近一次 ingest、query 写入或 lint 修复
- 最近 `log.md` 是否显示未完成动作

不要因为按需目录不存在就立即报错。只有当 raw-index、页面链接或日志显示应该存在时，才报缺失。

### 3. raw-index 一致性巡检

检查：

- `raw/library/sources/` 和 `raw/library/notes/` 中是否有文件未登记到 `raw-index.md`
- `raw-index.md` 是否登记了不存在的 raw 路径
- `raw-index.md` 是否登记了仍在 `raw/inbox/` 的文件
- `status` 是否只使用 `ingested`、`needs-update`
- `needs-update` 是否有 `update_reason`
- `ingested` 是否错误保留 `update_reason`
- `last_status_at` 是否符合 `YYYY-MM-DD HH:MM:SS`
- `wiki_page` 是否指向存在的 `wiki/sources/` 追踪页

缺失或错误必须列出具体 raw path / raw-index 行 / wiki_page。

### 4. 来源追踪页巡检

检查每个已登记 raw item：

- 是否有对应 `wiki/sources/<source-page>.md`
- source page frontmatter 是否包含：
  - `page_type: source`
  - `tags`
  - `created_at`
  - `updated_at`
  - `raw_path`
  - `raw_index_id`
  - `raw_bucket`
- `raw_bucket` 是否与 `raw_path` 所在 bucket 一致
- source page 是否有摘要、关键观点、证据 / 摘录
- source page 是否链接相关 `wiki/knowledge/` 或 `wiki/syntheses/`
- source page 是否只写“暂无关联”，但 raw 内容明显可抽取知识点

特别注意：`raw/library/notes/` 中的文件也必须有 `wiki/sources/` 追踪页；不要把 raw notes 直接当成 knowledge page。

### 5. knowledge 巡检

检查 `wiki/knowledge/`：

- frontmatter 是否包含 `page_type: knowledge`、`tags`、`created_at`、`updated_at`、`knowledge_type`
- 是否有当前理解、关键来源、证据、相关 knowledge、不确定或冲突、下一步
- 关键来源是否链接到 `wiki/sources/`
- 证据是否能回到 source page 或 raw
- 是否存在几乎重复的 knowledge page
- 是否存在稳定知识只散落在 source page，没有独立 knowledge page
- 是否存在 knowledge page 没有任何 source 支撑

发现缺失 knowledge 文档时，输出：

- 候选 knowledge 标题
- 支撑 source pages
- 为什么值得独立成页
- 建议动作：走 ingest 补同步，或走 query 写回

### 6. synthesis 巡检

检查 `wiki/syntheses/`：

- frontmatter 是否包含 `page_type: synthesis`、`tags`、`created_at`、`updated_at`、`synthesis_type`
- 是否有范围、当前结论、共识、分歧、证据链、缺口、相关 sources / knowledge
- 是否链接相关 source pages 和 knowledge pages
- 是否存在多个 source / knowledge 已经形成稳定综合方向，但没有 synthesis page
- 是否存在 synthesis 结论过强、证据不足或未反映分歧

发现缺失 synthesis 文档时，输出：

- 建议 synthesis 标题
- 相关 sources / knowledge
- 可综合方向
- 为什么当前适合或暂不适合创建
- 建议动作：query 写回 synthesis 或先补证据

### 7. 链接与孤儿页巡检

检查：

- `index.md` 是否有重要入口
- `index.md` 的“待处理”是否把已完成事项误写为待处理
- `index.md` 的“最近新增”是否漏掉最近的 ingest / query-writeback / lint-repair
- 页面是否缺少入链或出链
- source page 是否链接对应 knowledge / synthesis
- knowledge / synthesis 是否反向链接关键 source page
- 是否存在严格孤儿页：无导航入口、无入链、无明显出链
- 是否存在弱连接页：有导航但缺少上下文互链
- 是否存在断链 `[[...]]`

区分：

- 严格孤儿页
- 弱连接页
- 断链
- 缺少反向链接
- index 陈旧或待处理误归类

不要把已经在 `index.md` 有入口的页面误报为严格孤儿页。

### 8. 内容冲突巡检

检查：

- source page 之间是否有明确冲突
- knowledge page 是否与其关键来源不一致
- synthesis page 是否遗漏重要分歧
- synthesis page 是否把弱证据写成强结论
- 不同页面是否对同一概念使用了不同定义
- 是否存在陈旧结论被后续来源削弱

输出冲突文档时必须包含：

- 冲突类型：来源冲突 / 总结过强 / 定义不一致 / 证据不足 / 陈旧结论
- 冲突页面
- 冲突内容
- 证据位置
- 建议动作

不要只写“存在冲突”，也不要在没有证据时强行判定冲突。

### 9. needs-update 候选

可建议 `needs-update` 的情况：

- raw 文件修改时间明显晚于 `last_status_at`
- `wiki/sources/` 缺失或不符合 Page Shapes
- 关键 source page 与 raw-index 指向不一致
- 用户或日志明确标记来源需要刷新
- source page / knowledge page / synthesis page 的证据链断裂，需要重新同步

不要因为“有待跟进问题”就自动建议 `needs-update`。待跟进问题可以留在页面或建议 query。

### 10. 生成巡检报告

报告必须按问题类型组织，至少包含：

- 一致性问题
- 缺失文档
- 冲突文档
- 结构与链接问题
- frontmatter / Page Shapes 问题
- needs-update 候选
- 建议动作

没有问题的类别可以写“未发现明显问题”。

## 修复规则

只有用户明确要求修复时，才执行写操作。

允许的最小修复：

- 补缺失 frontmatter 字段
- 修正 `raw_bucket` 与 `raw_path` 不一致
- 补明显缺失的 `[[...]]` 双链
- 清空 `ingested` 状态下错误保留的 `update_reason`
- 为明确候选写入 `needs-update` 和 `update_reason`
- 更新 `index.md` 的高价值入口
- 清理 `index.md` 中已经完成但仍留在“待处理”的事项
- 将本次 lint 修复摘要追加到 `index.md` 的“最近新增”
- 在 `log.md` 追加 lint 修复记录

`index.md` 修复规则：

- “待处理”只保留尚未完成、需要后续 action 的事项。
- 已完成的补链、frontmatter 修复、schema 对齐、source / knowledge / synthesis 更新，不应继续放在“待处理”。
- 执行修复后，必须在“最近新增”追加一条简短记录，说明时间、动作类型和修复范围。
- “最近新增”使用真实当前本地时间，格式 `YYYY-MM-DD HH:MM:SS`，写入前必须读取时间。
- 不要把 `log.md` 的完整记录复制进 `index.md`；`index.md` 只写高价值摘要入口。

不允许 lint 自动执行：

- 新来源 ingest
- 移动 raw 文件
- 批量重写 wiki 内容
- 自动创建大量 knowledge / synthesis 页面
- 自动删除历史页面或目录
- 自动迁移 `wiki/notes/`

如果修复会变成 ingest 或 query 写回，应建议用户改用对应 skill。

## 输出格式

默认使用：

```md
一致性问题：
- [级别] 文件/页面：问题。证据：...。建议：...

缺失文档：
- [级别] 缺失：...。依据：...。建议动作：...

冲突文档：
- [级别] 冲突类型：...
  冲突页面：...
  冲突内容：...
  证据：...
  建议动作：...

结构与链接问题：
- ...

index.md 问题：
- ...

frontmatter / Page Shapes 问题：
- ...

needs-update 候选：
- ...

建议动作：
- ...
```

级别建议：

- `P0`：状态或证据链严重错误，影响可信度
- `P1`：缺失关键页面、冲突文档、明显断链
- `P2`：弱连接、frontmatter 不完整、建议补强
- `P3`：命名、标签、导航优化

如果已经执行修复，追加：

```md
已修复：
- ...

已更新：
- raw-index.md
- wiki/sources/...
- wiki/knowledge/...
- wiki/syntheses/...
- index.md
- log.md
```

## 不要这样做

- 不要把 lint 做成 init。
- 不要把 lint 做成 ingest。
- 不要把 lint 做成 query 或文章写作。
- 不要创建 `wiki/notes/`。
- 不要只给抽象评价，不列具体文件或页面。
- 不要发现缺失文档却不说缺什么。
- 不要发现冲突文档却不列冲突双方。
- 不要在用户未要求时静默修复。
- 不要把可疑信号写成已确认事实。
- 不要因为按需目录不存在就直接报错。

## 示例触发

**示例 1**
用户：`检查 raw-index 和 raw/library 是否一致。`

动作：使用本 skill，检查 raw-index 条目、raw 路径、wiki_page 和状态字段，输出具体不一致项。

**示例 2**
用户：`帮我找缺失的文档和冲突的文档。`

动作：使用本 skill，列出缺失 source / knowledge / synthesis 文档，以及冲突页面、冲突内容、证据和建议动作。

**示例 3**
用户：`哪些页面是孤儿页，哪些链接断了？`

动作：使用本 skill，区分严格孤儿页、弱连接页、断链和缺少反向链接。

**示例 4**
用户：`顺手把明显的 frontmatter 问题修一下并记日志。`

动作：使用本 skill，先审计，再执行最小修复，最后更新 `index.md` 和 `log.md`。
