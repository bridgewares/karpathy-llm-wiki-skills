# Raw 来源状态表

`raw-index.md` 是当前 vault 中 raw 来源 ingest 状态的唯一官方状态表。

## 规则
- 只有 `raw/library/` 中的来源才登记在这里。
- `raw/inbox/` 中的内容表示待处理，不在此表登记正式状态。
- 当前阶段暂不使用单文件 metadata，统一以本文件为准。
- 状态最少使用：`new`、`ingested`、`needs-update`。
- 当状态为 `needs-update` 时，必须填写 `update_reason`。
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。
- 现有自定义 taxonomy `interviews/` 继续保留；新增默认分类仅作为补充骨架。

## 字段建议
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## interviews
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## articles
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## videos
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## books
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## podcasts
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## webclips
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## pdfs
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |

## images
| source_id | title | type | status | added_at | updated_at | update_reason |
| --- | --- | --- | --- | --- | --- | --- |
