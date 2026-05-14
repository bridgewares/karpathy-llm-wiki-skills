# 工作流

## init
- 用于全新初始化或缺失补齐 wiki 骨架。
- 只创建目录、根文件、模板和 system 规则，不导入真实来源。

## ingest
1. 新来源进入 `raw/inbox/<type>/`。
2. 确认后移入 `raw/library/<type>/`。
3. 在 `raw-index.md` 登记或更新状态。
4. 创建或更新 `wiki/sources/<type>/` 来源页。
5. 抽取主题、人物、概念、问题并更新相关 wiki 页面。
6. 维护 `index.md` 与 `log.md`。

## query
1. 先读 `index.md`。
2. 优先查看相关 wiki 页面。
3. wiki 不足时再回到 raw。
4. 有长期价值的结论回写到 `wiki/questions/` 或 `wiki/syntheses/`。

## lint
- 检查 `raw/library/` 与 `raw-index.md` 是否一致。
- 检查来源页与主题页是否存在缺失链接或陈旧状态。
- 发现 `needs-update` 时必须附带 `update_reason`。
