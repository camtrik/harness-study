---
type: prompt
name: gsd:audit-fix
description: 自主审计到修复的流水线 — 发现问题、分类、修复、测试、提交
argument-hint: "--source <audit-uat> [--severity <medium|high|all>] [--max N] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Agent
  - AskUserQuestion
---
<objective>
运行审计，将发现的问题分类为可自动修复与仅可手动修复，然后自主修复可自动修复的问题，并进行测试验证和原子提交。

标志：
- `--max N` — 最多修复 N 个问题（默认：5）
- `--severity high|medium|all` — 要处理的最低严重级别（默认：medium）
- `--dry-run` — 仅对发现的问题进行分类而不修复（显示分类表格）
- `--source <audit>` — 运行哪个审计（默认：audit-uat）
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/audit-fix.md
</execution_context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/audit-fix.md 中的 audit-fix 工作流。
</process>
