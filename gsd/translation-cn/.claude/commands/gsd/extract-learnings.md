---
name: gsd:extract-learnings
description: 从已完成的阶段制品中提取决策、经验教训、模式和意外发现
argument-hint: <phase-number>
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - Agent
type: prompt
---
<objective>
从已完成的阶段制品（PLAN.md、SUMMARY.md、VERIFICATION.md、UAT.md、STATE.md）中提取结构化学习成果，写入 LEARNINGS.md 文件，涵盖决策、经验教训、发现的模式和遇到的意外情况。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/extract_learnings.md
</execution_context>

端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/extract_learnings.md 中的 extract-learnings 工作流。
