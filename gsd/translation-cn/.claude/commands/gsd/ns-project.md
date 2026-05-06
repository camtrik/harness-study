---
name: gsd-project
description: "项目生命周期 | 里程碑 审计 摘要"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的项目/里程碑 skill。
`gsd-plan-milestone-gaps` 在 #2790 中被删除 — 差距规划现在
作为 `gsd-audit-milestone` 输出的一部分内联进行。

| 用户想要 | 调用 |
|---|---|
| 开始新项目 | gsd-new-project |
| 创建新里程碑 | gsd-new-milestone |
| 完成当前里程碑 | gsd-complete-milestone |
| 审计里程碑问题 | gsd-audit-milestone |
| 总结里程碑状态 | gsd-milestone-summary |

使用 Skill 工具直接调用匹配的 skill。
