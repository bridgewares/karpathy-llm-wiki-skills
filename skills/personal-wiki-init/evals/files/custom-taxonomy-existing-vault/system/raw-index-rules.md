# raw-index 维护规则

`raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。

## 适用范围
- 仅登记 `raw/library/` 中的来源。
- `raw/inbox/` 只表示待处理区，不在 `raw-index.md` 中登记正式状态。

## 状态语义
- `new`：已进入 `raw/library/`，但尚未完成 ingest。
- `ingested`：已完成来源页整理，并同步到相关 wiki 页面。
- `needs-update`：来源已处理过，但原文变化、结构变化或结论需要刷新。

## 必填要求
- `needs-update` 必须填写 `update_reason`。
- 时间字段使用 `YYYY-MM-DD HH:MM:SS`。
- 一个正式来源只保留一个官方条目，避免重复登记。

## 维护动作
1. 来源从 inbox 转入 library 后，创建或更新条目。
2. ingest 完成后更新状态与时间。
3. 需要重做时将状态改为 `needs-update` 并写明原因。
