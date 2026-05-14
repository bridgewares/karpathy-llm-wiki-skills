# Raw Index 规则

`raw-index.md` 是唯一官方 raw ingest 状态表。

## 登记边界
- 只登记 `raw/library/` 中的来源。
- `raw/inbox/` 中的文件表示待处理，不提前登记。
- 当前阶段暂不使用单文件 metadata。

## 状态语义
- `new`：已进入 `raw/library/`，但尚未完成 ingest。
- `ingested`：已完成来源页和相关知识页更新。
- `needs-update`：来源或关联知识页需要重新检查或刷新。

## `needs-update` 要求
- 必须附带 `update_reason`。
- 更新后应同步刷新 `updated_at`。

## 时间字段
- `added_at`、`updated_at` 以及其他状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。
