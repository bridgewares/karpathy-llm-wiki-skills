# 工作流

## 初始化范围
- 初始化只负责目录、模板、规则文件与根导航。
- 不在初始化阶段伪造来源条目，不自动创建具体 source page。

## ingest 工作流
1. 把新来源放入 `raw/inbox/<type>/`。
2. 确认来源类型与命名。
3. 移动到 `raw/library/<type>/`。
4. 在 `raw-index.md` 登记状态。
5. 创建或更新 `wiki/sources/` 对应来源页。
6. 提取主题、概念、人物、问题并更新相关页面。
7. 更新 `index.md`。
8. 在 `log.md` 记录操作。

## query 工作流
1. 先查看 `index.md`。
2. 优先读相关 `wiki/` 页面。
3. 如果 wiki 信息不足，再回到 `raw/`。
4. 长期有价值的回答回写到 `wiki/questions/` 或 `wiki/syntheses/`。

## lint 工作流
1. 检查 `raw/library/` 与 `raw-index.md` 是否一致。
2. 检查来源页、主题页、概念页、人物页是否存在断链或遗漏。
3. 检查 `needs-update` 是否都附带 `update_reason`。
4. 检查时间字段是否统一为秒级格式 `YYYY-MM-DD HH:MM:SS`。
