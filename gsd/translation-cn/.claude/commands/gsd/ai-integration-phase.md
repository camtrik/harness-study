---
name: gsd:ai-integration-phase
description: 为涉及构建 AI 系统的阶段生成 AI-SPEC.md 设计合同
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - WebSearch
  - AskUserQuestion
  - mcp__context7__*
---
<objective>
为涉及 AI 系统开发的阶段创建 AI 设计合同（AI-SPEC.md）。
编排 gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner。
流程：选择框架 → 研究文档 → 研究领域 → 设计评估策略 → 完成
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ai-integration-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-frameworks.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-evals.md
</execution_context>

<context>
阶段编号：$ARGUMENTS — 可选，省略时自动检测下一个未计划的阶段。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ai-integration-phase.md。
保留所有工作流关卡。
</process>
