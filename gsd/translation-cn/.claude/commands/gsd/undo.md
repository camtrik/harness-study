---
name: gsd:undo
description: "安全的 git 回滚。使用阶段清单回滚阶段或计划提交，并包含依赖检查"
argument-hint: "--last N | --phase NN | --plan NN-MM"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
安全的 git 回滚 — 使用阶段清单回滚 GSD 阶段或计划提交，执行前包含依赖检查和确认关卡。

三种模式：
- **--last N**：显示最近的 GSD 提交以供交互选择
- **--phase NN**：回滚某个阶段的所有提交（清单 + git log 回退）
- **--plan NN-MM**：回滚特定计划的所有提交
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/undo.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/gate-prompts.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/undo.md 中的 undo 工作流。
</process>
