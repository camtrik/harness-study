---
name: gsd:cleanup
description: 归档已完成里程碑累积的阶段目录
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---
<objective>
将已完成里程碑的阶段目录归档到 `.planning/milestones/v{X.Y}-phases/`。

当 `.planning/phases/` 中累积了过往里程碑的目录时使用。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/cleanup.md
</execution_context>

<process>
按照 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/cleanup.md 中的清理工作流执行。
识别已完成的里程碑，展示试运行摘要，确认后归档。
</process>
