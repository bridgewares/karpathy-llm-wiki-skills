# Raw Index

`raw-index.md` 是当前唯一官方 raw ingest 状态表。

## 维护规则
- 当前阶段暂不使用单文件 metadata。
- 只有位于 `raw/library/` 中的来源才在这里登记。
- `raw/inbox/` 中的内容表示待处理，不应在此提前登记。
- 状态至少使用：`new`、`ingested`、`needs-update`。
- 若状态为 `needs-update`，必须填写 `update_reason`。
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。

## 建议字段
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## articles
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## videos
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## books
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## podcasts
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## webclips
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## pdfs
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|

## images
| source_id | title | type | path | status | added_at | updated_at | update_reason |
|---|---|---|---|---|---|---|---|
