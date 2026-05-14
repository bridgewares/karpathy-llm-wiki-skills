# 工作流

## ingest
1. 新来源先进入 `raw/inbox/<type>/`。
2. 确认来源类型与文件命名。
3. 移动或整理到 `raw/library/<type>/`。
4. 在 `raw-index.md` 中登记或更新状态。
5. 创建或更新 `wiki/sources/` 来源页。
6. 提取主题、概念、人物、问题并更新相关 wiki 页面。
7. 更新 `index.md` 与 `log.md`。

## query
1. 先查看 `index.md`。
2. 优先读取相关 `wiki/` 页面。
3. wiki 信息不足时再回到 `raw/`。
4. 产生长期有价值的总结时，回写到 `wiki/questions/` 或 `wiki/syntheses/`。

## lint
1. 检查 `raw/library/` 与 `raw-index.md` 是否一致。
2. 检查来源页与知识页是否存在明显缺链、冲突或陈旧状态。
3. 将发现记录到 `log.md`，必要时标记 `needs-update`。
