---
name: gsd:import
description: 导入外部计划，在写入任何内容之前检测与项目决策的冲突
argument-hint: "--from <filepath>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
---

<objective>
将外部计划文件导入 GSD 规划系统，并与 PROJECT.md 的决策进行冲突检测。

- **--from**：导入外部计划文件，检测冲突，写入 GSD PLAN.md，通过 gsd-plan-checker 验证。

将来：`--prd` 模式用于 PRD 提取，计划在后续 PR 中实现。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/import.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/gate-prompts.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/doc-conflict-engine.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
端到端执行 import 工作流。
</process>
