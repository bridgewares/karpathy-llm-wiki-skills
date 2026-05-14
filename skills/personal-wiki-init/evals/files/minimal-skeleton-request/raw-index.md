# Raw 来源索引

`raw-index.md` 是当前唯一官方 raw ingest 状态表。

## 维护规则
- 当前阶段暂不使用单文件 metadata 作为官方 ingest 状态来源。
- 只有进入 `raw/library/` 的来源才应登记在这里。
- `raw/inbox/` 中的内容表示待处理，不应直接视为已 ingest。
- 状态至少使用：`new`、`ingested`、`needs-update`。
- 当状态为 `needs-update` 时，必须填写 `update_reason`。
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。

## 建议字段
| source_id | type | path | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## 按类型登记骨架

### articles
| source_id | path | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- |

### videos
| source_id | path | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- |

### books
| source_id | path | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- |

### webclips
| source_id | path | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- |
