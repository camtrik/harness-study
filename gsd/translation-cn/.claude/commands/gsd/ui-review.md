---
name: gsd:ui-review
description: 对已实现的前端代码进行回溯性 6 支柱可视化审计
argument-hint: "[phase]"
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
进行回溯性 6 支柱可视化审计。生成 UI-REVIEW.md，
包含分级评估（每支柱 1-4 分）。适用于任何项目。
输出：{phase_num}-UI-REVIEW.md
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ui-review.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
阶段：$ARGUMENTS — 可选，默认为最后完成的阶段。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ui-review.md。
保留所有工作流关卡。
</process>
