---
name: gsd:ui-phase
description: 为前端阶段生成 UI 设计合同（UI-SPEC.md）
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - AskUserQuestion
  - mcp__context7__*
---
<objective>
为前端阶段创建 UI 设计合同（UI-SPEC.md）。
编排 gsd-ui-researcher 和 gsd-ui-checker。
流程：验证 → 研究 UI → 验证 UI-SPEC → 完成
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ui-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
阶段编号：$ARGUMENTS — 可选，省略时自动检测下一个未计划的阶段。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ui-phase.md。
保留所有工作流关卡。
</process>
