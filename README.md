# Personal Wiki Skills

一组围绕个人 LLM Wiki / Obsidian-first 知识库工作流设计的项目级 skills，目标是把“初始化骨架、来源入库、知识查询、健康巡检”拆成边界清晰、可组合的四条主流程。

当前包含：

- `personal-wiki-init`：初始化或补齐 wiki 骨架
- `personal-wiki-ingest`：把具体来源从 `raw/inbox/` / `raw/library/` 纳入正式知识流
- `personal-wiki-query`：基于现有 wiki 内容回答、比较、综合与必要的证据补查
- `personal-wiki-lint`：做一致性、结构与内容巡检

## 设计目标

这组 skills 不是在做“通用笔记助手”，而是在约束一个相对明确的个人 wiki 工作模型：

1. **`raw` 与 `wiki` 分层**
   - `raw/` 保存原始资料
   - `wiki/` 保存结构化知识页

2. **`raw/inbox` 与 `raw/library` 分层**
   - `raw/inbox/` 是待处理入口
   - `raw/library/` 是正式原始资料库

3. **`raw-index.md` 统一追踪来源状态**
   - 用于登记来源是否已入库、已 ingest、是否需要更新

4. **围绕工作流拆 skill，而不是围绕文件类型拆 skill**
   - init / ingest / query / lint 分别对应不同意图
   - 减少“一个 skill 什么都做”的边界污染

## 适用人群

适合以下场景：

- 你在维护一个个人知识库 / 研究型 wiki
- 你希望 LLM 协助处理原始资料、来源页与知识页
- 你希望把“整理资料”“回答问题”“巡检一致性”拆成不同流程
- 你接受较强约束：目录分层、状态登记、结构化页面、最小但明确的流程边界

如果你只是想做一个非常自由的 Obsidian 笔记仓库，这套 skills 可能会显得偏重。

---

## Skills 概览

### 1. `personal-wiki-init`

用于：

- 新建一个个人 wiki
- 给已有但不完整的 vault 补齐骨架
- 补目录、根文件、system 规则文件

核心职责：

- 建立或补齐 `raw/`、`wiki/`、`system/`
- 建立 `CLAUDE.md`、`raw-index.md`、`index.md`、`log.md`、`inbox.md`
- 明确 `raw` / `wiki` 边界与 `raw-index.md` 规则

不负责：

- 导入具体来源
- 回答知识问题
- 做巡检或一致性审计

### 2. `personal-wiki-ingest`

用于：

- 首次 ingest 一个来源
- 批量 ingest 多个待处理来源
- 对已有来源 re-ingest / refresh / update

核心职责：

- 识别目标来源及其类型
- 整理到 `raw/library/`
- 维护 `raw-index.md`
- 创建或更新 `wiki/sources/...` 来源页
- 抽取结构化属性
- 自动同步相关 `wiki/notes/...` 页面，必要时更新 `wiki/syntheses/...`
- 更新 `index.md` 与 `log.md`

不负责：

- 初始化整个 wiki 骨架
- 纯问答 / 纯总结
- 巡检孤儿页、交叉引用或 `needs-update` 全局审计

### 3. `personal-wiki-query`

用于：

- 基于现有 wiki 回答问题
- 比较多个页面或多个来源
- 做阶段性总结或综合
- 必要时回到 `raw/library/` 补证据

核心职责：

- 优先复用 `wiki/` 层，而不是默认从 raw 起步
- 输出结论、依据、冲突与空白
- 在高价值场景下建议写回 `wiki/notes/` 或 `wiki/syntheses/`
- 若用户明确要求落盘，则直接写回并更新 `log.md`

不负责：

- 新来源入库
- 维护 `raw-index.md`
- 巡检一致性
- 没有 query 意图的普通单页改写

### 4. `personal-wiki-lint`

用于：

- 巡检 `raw/library/`、`raw-index.md`、`wiki/sources/` 一致性
- 查孤儿页、弱连接页、缺失交叉引用、命名不一致
- 识别陈旧内容、证据缺口、应标 `needs-update` 的来源
- 盘点哪些实体值得拆成独立页面

核心职责：

- 默认先审计和汇报
- 只有在用户明确要求时才顺手修复
- 把问题分类为一致性 / 结构 / 内容 / 建议动作

不负责：

- 新来源 ingest
- 知识问答或专题综合
- 初始化骨架

---

## 推荐工作流

这四个 skills 组合起来，大致对应下面的闭环：

### A. 先搭骨架

使用 `personal-wiki-init`：

- 新建目录结构
- 明确规则文件
- 准备 workflow Page Shapes
- 建立状态追踪基础

### B. 再处理来源

使用 `personal-wiki-ingest`：

- 把新资料从 `raw/inbox/` 处理进 `raw/library/`
- 登记到 `raw-index.md`
- 生成来源页并同步知识页

### C. 基于已有知识回答问题

使用 `personal-wiki-query`：

- 优先看 `wiki/`
- 不够时再回到 `raw/library/`
- 必要时把高价值结果沉淀回 wiki

### D. 定期做健康检查

使用 `personal-wiki-lint`：

- 查结构断裂
- 查状态不一致
- 查陈旧结论与证据缺口
- 给出下一步维护建议

一个典型循环是：

`init → ingest → query → lint → 再 ingest / 再 query`

---

## 目录与模型假设

这组 skills 默认假设你的个人 wiki 大致采用如下结构：

```text
raw/
  inbox/
  library/
wiki/
raw-index.md
index.md
log.md
CLAUDE.md
```

其中：

- `raw-index.md` 是来源状态表
- `wiki/` 是 LLM 维护的知识层根目录
- `raw/inbox/` 和 `raw/library/` 只按需创建 `sources/`、`notes/`
- `wiki/` 只按需创建 `sources/`、`notes/`、`syntheses/`
- 主题、人物、概念、问题等细分放在页面 frontmatter、标签或正文小节中，不作为默认目录名

---

## 目录约束

这组 skills 采用固定的最小目录模型：

- `init` 只建立最小可用骨架，默认保留 `raw/inbox/`、`raw/library/` 和 `wiki/` 这些稳定入口，不强制预建大量 raw 或 wiki 子目录。
- `ingest` 可以把来源从 inbox 整理到 library，但 raw 来源正文仍是事实源，不应被改写；移动后也要保留 inbox bucket 目录，保证后续入口稳定。需要创建 source/note/synthesis 页面时，再创建对应 wiki 子目录。
- `query` 的高价值答案可以写回 wiki；如果用户是在上一轮 query 后二次确认写回，应复用上一轮已确认答案，而不是重新生成一份可能不一致的文档。需要写回 notes/syntheses 时，再创建对应 wiki 子目录。

目录分类统一收敛为 raw 的 `sources / notes` 与 wiki 的 `sources / notes / syntheses`，避免 LLM 在细分目录之间做不必要的分类判断。

---

## 这套设计为什么合理

整体上，这组 skills 的设计是**合理且方向正确**的，原因主要有四点：

### 1. 按用户意图拆 skill，而不是按文件操作拆

这是最大的优点。

很多知识库自动化容易失败，是因为“导入资料、回答问题、巡检结构、改写页面”混在一个技能里，结果触发条件模糊，执行边界失控。这里把 init / ingest / query / lint 分开，能明显减少误触发和职责串线。

### 2. 明确把 `raw-index.md` 作为官方状态层

这让 ingest 与 lint 都有一个统一、稳定的参照面。对个人 wiki 而言，这比把状态散落在多个页面 frontmatter 里更容易审计和批处理，也更适合后续让 LLM 读写。

### 3. 强调“来源页是中间层”

`personal-wiki-ingest` 不是只移动文件，而是先沉淀 `wiki/sources/...` 的结构化属性，再同步到 `wiki/notes/...` 或 `wiki/syntheses/...`。这种设计能减少知识直接散落在多个下游页里，整体是对的。

### 4. query 与 lint 的边界意识比较强

`query` 强调优先使用 wiki、必要时再回 raw；`lint` 强调先审计、再决定是否修复。这两个约束都很关键，能避免 skill 一上来就“重做 ingest”或者“看见问题就改一堆”。

---

## 当前设计里的主要优点

### 优点 1：触发描述写得足够贴近自然语言

每个 skill 的 description 都尽量覆盖了用户真实会说的话，而不是只写抽象术语。这会提高触发准确率。

### 优点 2：边界反例写得比较清楚

每个 skill 都写了“不要用于什么”，这对减少误触发很有帮助，尤其是：

- ingest 不吞 query / lint
- query 不吞 ingest / 单页编辑
- lint 不吞 init / query

### 优点 3：evals 覆盖了关键主路径

从你现在的 evals 看，已经覆盖了不少关键场景：

- init：空目录、补齐、保留现有内容、最小骨架
- ingest：首次 ingest、re-ingest、跨页同步、批量 ingest
- query：直接问答、跨页比较、wiki-first raw fallback、建议写回、显式写回、边界负例

这说明这套 skills 不是只停留在文案层，而是在尝试做行为约束。

---

## 目前最值得改进的地方

下面这些不是方向性错误，而是会影响长期可维护性和触发稳定性的点。

### 1. `init` 应始终保持最小骨架

问题：

- 如果 init 创建过多目录，会诱导后续流程填充不存在的分类需求
- 如果 init 预建 bucket 子目录，会削弱“行为触发创建”的约束

建议：

- init 只创建 `raw/inbox/`、`raw/library/`、`wiki/`、`system/`
- raw bucket 和 wiki bucket 都由 ingest/query 首次需要时创建
- eval 应持续检查 init 不预建 bucket 子目录

### 2. `ingest` 的职责已经接近“半自动知识建模器”

现在的 `personal-wiki-ingest` 很强：不仅建 source page，还要自动更新 `wiki/notes/...`，必要时还会更新 `wiki/syntheses/...`。

这很有价值，但也最容易失控。

风险：

- 单次 ingest 改动面过大
- 对信息密度高的来源，可能制造过多新页面
- 触发时若未限制“优先更新已有页”，容易出现碎片页泛滥

建议：

- 进一步强调“默认优先更新已有页面，新建页面需要达到更高阈值”
- 把“自动同步”分成两层：
  - 必做：来源页 + `raw-index.md` + `log.md`
  - 条件做：`wiki/notes/...` 或 `wiki/syntheses/...` 页面更新
- 在 skill 中补一个“当来源过宽时，只抽取来源页结构化属性，不强制全部下游拆页”的显式规则

### 3. `query` 与“普通页面编辑”之间仍有轻微灰区

你已经写了负边界，但还有一个实际问题：

- 用户说“把这个问题当前答案整理一下并写回 note 页”时，既像 query，也像编辑
- 用户说“更新一下 Agent Memory 这一页，顺便补当前结论”时，也可能落在 query / edit 中间

建议：

- 在 README 和 skill 中明确：**query 的判定依据是“是否围绕一个提问、比较、核对、总结动作”**，而不是“最终是否会写页面”
- 如果主要目标是“回答一个问题并可落盘”，归 query
- 如果主要目标是“直接改一个页面表达”，不归 query

### 4. `lint` 的结果分流可以再明确

现在 lint 已经会输出建议动作，但可以更进一步标准化：

- 哪类问题建议走 ingest
- 哪类问题建议走 query
- 哪类问题适合直接最小修复

建议：

在 lint 输出规范里加一类字段，例如：

- `建议工作流`：`ingest / query / lint-fix / manual-review`

这样 lint 更像“调度与审计层”，而不是单纯问题列表。

### 5. 四个 skill 的共享约束还没有统一抽到公共文档

现在很多关键规则在多个 skill 里重复出现，例如：

- 中文优先
- `raw-index.md` 是唯一官方状态表
- `needs-update` 必须带 `update_reason`
- 不把猜测写成事实
- 尽量最小修改

问题：

- 重复维护成本高
- 文案一旦漂移，不同 skill 可能出现细微冲突

建议：

- 在这组项目级 skills 下补一个共享约定文档，类似：`skills/personal-wiki-shared/` 或项目根 README 中的“统一约束”章节
- 各 skill 只保留本流程特有规则，共享规则以引用方式维护

---

## 对触发与边界的建议

如果你想继续提高这套 skills 的稳定性，我建议重点增强下面三件事。

### 建议 1：把“单页编辑”单独抽成辅助 skill 或明确留给通用编辑流

当前四个 skill 没有专门覆盖“仅编辑一个现有 wiki 页面”的情况。

这会导致：

- query 容易被误触发去吞页面改写
- lint 也可能误吸收“顺手修一页”的请求

两种做法都可以：

- 新增一个 `personal-wiki-edit-page`
- 或在 README 中明确：单页表达改写不属于这四个主 skill，交给通用编辑流程

### 建议 2：给 `ingest` 增加“保守模式”定义

例如：

- **保守 ingest**：只做来源入库、source page、raw-index、log
- **扩展 ingest**：再做下游 `wiki/notes/...` 或 `wiki/syntheses/...` 同步

这样能更适合不同用户偏好，也便于后续 eval 分层。

### 建议 3：给 `query` 增加“答案级别”概念

例如：

- 快速回答：只基于现有 wiki 页
- 核对回答：允许回 raw 补证据
- 写回答案：回答后直接沉淀回 wiki

这会让 query 的输出更稳定，也更便于测试。

---

## 推荐的仓库组织方式

如果你准备继续维护这套 skills，建议仓库逐渐整理成下面这种结构：

```text
skills/
  personal-wiki-init/
    SKILL.md
    evals/
  personal-wiki-ingest/
    SKILL.md
    evals/
  personal-wiki-query/
    SKILL.md
    evals/
  personal-wiki-lint/
    SKILL.md
    evals/
README.md
```

并在根 README 中说明：

- 整体定位
- 四个 skill 的边界
- 推荐工作流
- 已知限制
- 后续扩展方向

---

## 已知限制

这套 skills 当前更适合：

- 单用户维护的个人 wiki
- 明确接受 `raw-index.md` 作为状态中心的工作流
- 愿意让 LLM 参与来源结构化提取与跨页同步的人

不太适合：

- 完全自由、无固定目录模型的笔记仓库
- 多人协作且规则持续变化很快的 wiki
- 极端依赖 frontmatter / Dataview 查询而不想维护 `raw-index.md` 的体系

---

## 后续演进建议

优先级从高到低，我建议是：

1. **先补根 README**
   - 把四个 skill 的定位和边界统一说明清楚

2. **再收敛共享规则**
   - 减少 skill 文案重复

3. **再补单页编辑的边界策略**
   - 决定是新增 skill，还是明确交给通用编辑流

4. **最后再细化 ingest / query 的模式分层**
   - 保守 / 扩展 ingest
   - 快速 / 核对 / 写回答案 query

---

## 快速示例

### 初始化

- “帮我在这个目录初始化一个个人 LLM wiki”
- “这个 vault 不完整，补齐成标准结构”

→ 使用 `personal-wiki-init`

### 来源入库

- “把这篇文章 ingest 到 wiki”
- “处理 raw/inbox/sources 里的几个 webclip 文件”
- “这个来源原文更新了，重新 ingest”

→ 使用 `personal-wiki-ingest`

### 基于 wiki 回答问题

- “这个 wiki 里现在怎么回答这个问题？”
- “比较几份关于 agent memory 的来源”
- “先基于现有 wiki 回答，不够再回原文”

→ 使用 `personal-wiki-query`

### 巡检与审计

- “检查哪些页面是孤儿页”
- “看下 raw-index 和 raw/library 是否一致”
- “哪些来源应该标 needs-update？”

→ 使用 `personal-wiki-lint`

---

## 总结

如果把目标定义为：**为个人 LLM Wiki 建立一套清晰、可执行、可评估的项目级工作流 skills**，那么你这套设计整体是成立的，而且已经比很多“只会写 prompt、不管边界”的 skill 设计成熟很多。

最核心的价值不在于它能“自动做很多事”，而在于它已经开始把个人 wiki 工作流拆成：

- 初始化
- 来源入库
- 知识查询
- 健康巡检

这四类稳定意图。

接下来最值得做的，不是推翻重来，而是：

- 收敛共享规则
- 进一步缩小灰区
- 给不同强度的 ingest / query 增加模式分层
- 明确单页编辑如何归位
