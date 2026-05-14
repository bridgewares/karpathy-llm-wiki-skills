# 自定义分类说明

这个 vault 目前在 raw 层使用一套自定义 taxonomy，而不是默认的文章 / 视频 / 书籍分类。

## 当前分类原则
- `interviews/` 用来存放访谈、播客对谈、AMA 记录、招聘访谈纪要等材料。
- 这类来源在后续分析里会被当成同一类来源处理，所以不要拆散到 `articles/` 或 `videos/`。
- 现有分类是长期使用的习惯，不是临时占位结构。

## 维护要求
- 如果骨架不完整，可以补齐缺失目录、根文件、模板和 system 规则文件。
- 不要因为初始化而重命名或删除 `raw/inbox/interviews/` 和 `raw/library/interviews/`。
- 如果需要补充默认分类，请保留现有 taxonomy，不要把已有 interviews 内容迁移走。
