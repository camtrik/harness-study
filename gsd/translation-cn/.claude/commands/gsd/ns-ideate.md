---
name: gsd-ideate
description: "探索捕获 | 探索 草图 spike spec 捕获"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的探索/捕获 skill。
`gsd-note`、`gsd-add-todo`、`gsd-add-backlog` 和 `gsd-plant-seed` 在
#2790 中被合并到 `gsd-capture`（分别使用 `--note`、默认、`--backlog`、`--seed` 模式）。
capture 目标通过 `--list` 列出待处理待办事项。

| 用户想要 | 调用 |
|---|---|
| 探讨一个想法或机会 | gsd-explore |
| 勾勒粗略的设计或计划 | gsd-sketch |
| 限时技术 spike | gsd-spike |
| 为阶段编写 spec | gsd-spec-phase |
| 捕获一个想法（待办/笔记/backlog/种子） | gsd-capture |

使用 Skill 工具直接调用匹配的 skill。
