# 工作流

## ingest
1. 新来源先进入 `raw/inbox/<type>/`。
2. 确认后整理到 `raw/library/<type>/`。
3. 在 `raw-index.md` 建立或更新条目。
4. 创建或更新对应 `wiki/sources/` 来源页。
5. 视内容补充相关 topic / concept / person / question 页面。
6. 更新 `index.md` 与 `log.md`。

## query
1. 先查看 `index.md`。
2. 优先读取相关 `wiki/` 页面。
3. 只有 wiki 信息不足时再回到 `raw/`。
4. 若产出长期有价值的总结，回写到 `wiki/questions/` 或 `wiki/syntheses/`。

## lint
1. 检查 `raw/library/` 与 `raw-index.md` 是否一致。
2. 检查来源页与主题页的交叉链接是否缺失。
3. 识别需要标记为 `needs-update` 的来源并说明 `update_reason`。
4. 将结果记录到 `log.md`。
