---
name: gsd-context
description: "代码库智能 | 地图 图谱 文档 学习"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的代码库智能 skill。
`gsd-scan` 和 `gsd-intel` 在 #2790 中被合并到 `gsd-map-codebase` 的标志中。

| 用户想要 | 调用 |
|---|---|
| 映射完整代码库结构 | gsd-map-codebase |
| 快速轻量级代码库扫描 | gsd-map-codebase --fast |
| 查询映射的智能文件 | gsd-map-codebase --query |
| 生成知识图谱 | gsd-graphify |
| 更新项目文档 | gsd-docs-update |
| 从已完成阶段提取学习成果 | gsd-extract-learnings |

使用 Skill 工具直接调用匹配的 skill。
