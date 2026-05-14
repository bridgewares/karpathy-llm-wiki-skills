# Raw Index 规则

## 角色
- `raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。
- 只有 `raw/library/` 中的来源才应在 `raw-index.md` 中登记。
- `raw/inbox/` 中的材料表示待处理，不代表已正式纳入管理。

## 状态语义
- `new`：来源已进入 `raw/library/`，但尚未完成结构化处理。
- `ingested`：来源已完成当前轮 ingest，并已同步对应 wiki 页面。
- `needs-update`：来源内容、结构或相关页面发生变化，需要重新处理。

## 必填要求
- 所有状态记录都应带秒级时间字段。
- 当状态为 `needs-update` 时，必须填写 `update_reason`。
- 不要把单文件 metadata 视为官方状态来源。
