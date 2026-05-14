# 个人 LLM Wiki 维护规则

## 1. 目标
这是一个由 LLM 协助维护的个人 wiki，用于持续积累来源、问题与知识页，而不是每次都从原始资料重新开始。

## 2. 目录边界
- `raw/` 保存原始资料。
- `raw/inbox/` 是新来源入口，按当前 taxonomy 放入对应子目录。
- `raw/library/` 是正式原始资料库，确认后从 inbox 移入。
- `raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。
- `wiki/` 保存整理后的知识页，不与 raw 混用。
- `templates/` 保存最小可用模板。
- `system/` 保存工作流、taxonomy、命名和 raw-index 规则。

## 3. 自定义 taxonomy 约定
- 当前 vault 已有自定义 raw 分类：`interviews/`。
- 初始化或补齐时必须保留 `raw/inbox/interviews/` 与 `raw/library/interviews/`。
- 如需新增默认分类，只能补充，不能替换或重写已有 taxonomy。

## 4. ingest 工作流
1. 新来源先进入 `raw/inbox/<type>/`。
2. 确认后移动到 `raw/library/<type>/`。
3. 在 `raw-index.md` 建立或更新条目。
4. 在 `wiki/sources/<type>/` 创建或更新来源页。
5. 按需要更新 `wiki/topics/`、`wiki/people/`、`wiki/concepts/`、`wiki/questions/` 等页面。
6. 每次 ingest 后更新 `index.md` 与 `log.md`。

## 5. query 与回写
1. 先看 `index.md`。
2. 优先读取相关 `wiki/` 页面。
3. 只有 wiki 信息不足时再回到 `raw/`。
4. 长期有价值的回答应回写到 `wiki/questions/` 或 `wiki/syntheses/`。

## 6. 页面创建与更新原则
- 优先更新已有页面，而不是重复建页。
- 只有值得长期维护的主题、人物、概念、项目或问题才单独建页。
- 初始化阶段只补骨架，不伪造来源条目，不批量创建空知识页。
- 发现信息不足时，先记录在来源页或 inbox，而不是强行扩张结构。

## 7. `index.md` 与 `log.md`
- `index.md` 用作高价值导航入口。
- `log.md` 记录 init、ingest、query、lint 等操作。
- 每次重要维护后都应追加记录，便于后续检索。

## 8. 写作风格
- 主要使用中文，保留必要英文术语。
- 区分结论、证据与未解决问题。
- 不把猜测写成事实。

## 9. 时间格式
- 所有用于状态判断的时间字段必须精确到秒。
- 统一使用 `YYYY-MM-DD HH:MM:SS`。
