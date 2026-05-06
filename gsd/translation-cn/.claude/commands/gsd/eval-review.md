---
name: gsd:eval-review
description: 审计已执行 AI 阶段的评估覆盖率，并生成 EVAL-REVIEW.md 修复计划
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---
<objective>
对已完成的 AI 阶段进行回溯性评估覆盖率审计。
检查 AI-SPEC.md 中的评估策略是否已实施。
生成 EVAL-REVIEW.md，包含得分、裁决、差距和修复计划。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/eval-review.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-evals.md
</execution_context>

<context>
阶段：$ARGUMENTS — 可选，默认为最后完成的阶段。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/eval-review.md。
保留所有工作流关卡。
</process>
