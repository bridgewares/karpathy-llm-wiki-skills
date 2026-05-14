# raw-index 规则

`raw-index.md` 是唯一官方 raw ingest 状态表。

## 适用范围
- 只登记 `raw/library/` 中的来源。
- `raw/inbox/` 中的资料表示待处理，不在官方索引中占正式条目。

## 状态语义
- `new`：已进入 `raw/library/`，但尚未完成结构化整理。
- `ingested`：已完成基础整理，并已有对应来源页或相关知识页更新。
- `needs-update`：来源或整理结果需要刷新，必须填写 `update_reason`。

## 维护要求
- 每次状态变化都更新时间。
- 不伪造条目；没有真实来源时保持空骨架即可。
