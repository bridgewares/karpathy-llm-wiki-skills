# Raw Index 维护规则

## 唯一性
- `raw-index.md` 是 raw 来源 ingest 状态的唯一官方状态表。
- 当前阶段不使用单文件 metadata 作为正式状态来源。

## 登记范围
- 只有 `raw/library/` 中的来源才允许登记到 `raw-index.md`。
- `raw/inbox/` 中的资料只表示待处理，不应提前登记。

## 状态语义
- `new`：已进入 `raw/library/`，但尚未完成 ingest。
- `ingested`：已完成来源页整理，并已把关键信息同步到相关 wiki 页面。
- `needs-update`：来源内容或关联知识页需要重新检查或重做。

## 必填要求
- `needs-update` 必须带 `update_reason`，不得省略。
- 所有状态时间字段必须使用 `YYYY-MM-DD HH:MM:SS`。
- 推荐保留稳定 `id` 与 `path` 字段，便于后续重建与追踪。

## 维护动作
1. 来源进入 `raw/library/` 后新增条目。
2. 状态变化时同步更新时间字段。
3. 如果标记为 `needs-update`，同时填写 `update_reason`。
4. 相关来源页重做后，再把状态改回 `ingested` 或其他适当状态。
