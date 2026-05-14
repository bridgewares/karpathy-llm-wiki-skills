# Raw Index

`raw-index.md` 是当前唯一官方 raw ingest 状态表。

## 规则
- 当前阶段暂不使用单文件 metadata。
- 只有 `raw/library/` 中的来源才登记到本表。
- `raw/inbox/` 中的内容表示待处理，不应在本表建正式条目。
- 状态至少使用：`new`、`ingested`、`needs-update`。
- `needs-update` 条目必须填写 `update_reason`。
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。

## 建议字段
- `source_id`
- `title`
- `type`
- `status`
- `added_at`
- `updated_at`
- `update_reason`
- `notes`

## articles
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## videos
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## books
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## podcasts
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## webclips
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## pdfs
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |

## images
| source_id | title | status | added_at | updated_at | update_reason | notes |
| --- | --- | --- | --- | --- | --- | --- |
