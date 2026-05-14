# Raw Index

`raw-index.md` 是当前 vault 中 raw 来源 ingest 状态的唯一官方状态表。

## 规则摘要
- 只登记 `raw/library/` 中的来源。
- `raw/inbox/` 中的资料不在这里登记，它们只表示待处理。
- 当前阶段暂不使用单文件 metadata，统一以本表追踪状态。
- 状态字段至少包括：`new`、`ingested`、`needs-update`。
- 当状态为 `needs-update` 时，必须填写 `update_reason`，不能留空。
- 所有状态时间字段必须精确到秒，统一使用 `YYYY-MM-DD HH:MM:SS`。

## 推荐字段
- `id`：稳定来源标识
- `title`：来源标题
- `type`：来源类型
- `path`：`raw/library/` 下的相对路径
- `status`：`new` / `ingested` / `needs-update`
- `status_time`：最近一次状态确认时间，格式 `YYYY-MM-DD HH:MM:SS`
- `update_reason`：仅在 `needs-update` 时必填
- `source_page`：对应 `wiki/sources/` 页面
- `notes`：必要备注

## 空白骨架

### articles
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### videos
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### books
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### podcasts
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### webclips
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### pdfs
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### images
| id | title | type | path | status | status_time | update_reason | source_page | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
