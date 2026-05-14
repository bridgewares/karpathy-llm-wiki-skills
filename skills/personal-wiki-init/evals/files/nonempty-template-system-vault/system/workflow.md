# Ingest / Query / Lint 工作流

## Ingest
1. 将新来源放入 `raw/inbox/<type>/`。
2. 确认来源类型与命名后，移动到 `raw/library/<type>/`。
3. 在 `raw-index.md` 登记来源及状态。
4. 创建或更新对应 `wiki/sources/` 来源页。
5. 抽取主题、概念、人物和问题，按需更新知识页。
6. 更新 `index.md` 与 `log.md`。

## Query
1. 先读 `index.md`。
2. 优先查看已有 `wiki/` 页面。
3. 仅在 wiki 信息不足时回到 `raw/` 原文。
4. 若形成稳定结论，可沉淀到 `wiki/questions/` 或 `wiki/syntheses/`。

## Lint
1. 检查 `raw/library/` 与 `raw-index.md` 是否一致。
2. 检查来源页与知识页是否存在缺失链接或明显陈旧内容。
3. 必要时记录待修复项，并在处理后更新 `log.md`。
