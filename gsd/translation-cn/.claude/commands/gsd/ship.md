---
name: gsd:ship
description: 验证通过后创建 PR、运行审查并准备合并
argument-hint: "[phase number or milestone, e.g., '4' or 'v1.0']"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - Write
  - AskUserQuestion
---
<objective>
桥接本地完成 → 已合并 PR。在 /gsd-verify-work 通过后，交付工作成果：推送分支，创建带有自动生成描述的 PR，可选择性地触发审查，并跟踪合并。

闭合 plan → execute → verify → ship 循环。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ship.md
</execution_context>

端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ship.md 中的 ship 工作流。
