# Raw Index 维护规则

`raw-index.md` 是唯一官方 raw ingest 状态表。

## 登记边界
- 只登记 `raw/library/` 中的来源。
- `raw/inbox/` 中的待处理来源不建正式条目。
- 当前阶段暂不依赖单文件 metadata。

## 状态语义
- `new`：已进入 `raw/library/`，但尚未完成结构化 ingest。
- `ingested`：已完成基础 ingest，并已有对应来源页或相关知识沉淀。
- `needs-update`：来源内容或结构变化，需要重新处理。

## needs-update 要求
- 每个 `needs-update` 条目必须填写 `update_reason`。
- 完成重处理后应更新时间字段并改回合适状态。

## 时间格式
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。
