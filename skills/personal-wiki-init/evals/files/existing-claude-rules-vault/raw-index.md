# Raw Index

`raw-index.md` 是 `raw/` 来源 ingest 状态的唯一官方状态表。

## 维护规则
- 只有已经进入 `raw/library/` 的来源才在这里登记。
- `raw/inbox/` 只作为待处理入口，不在本表登记正式状态。
- 当前阶段暂不使用单文件 metadata 作为官方状态来源。
- 状态至少使用：`new`、`ingested`、`needs-update`。
- `needs-update` 条目必须填写 `update_reason`。
- 所有状态时间字段统一使用 `YYYY-MM-DD HH:MM:SS`。

## 字段建议
- `source_id`：稳定来源标识
- `type`：来源类型
- `path`：`raw/library/` 中的相对路径
- `status`：`new` / `ingested` / `needs-update`
- `created_at`：首次登记时间
- `updated_at`：最近状态更新时间
- `update_reason`：仅 `needs-update` 必填
- `notes`：必要补充说明

## articles
- 暂无条目

## videos
- 暂无条目

## books
- 暂无条目

## podcasts
- 暂无条目

## webclips
- 暂无条目

## pdfs
- 暂无条目

## images
- 暂无条目
