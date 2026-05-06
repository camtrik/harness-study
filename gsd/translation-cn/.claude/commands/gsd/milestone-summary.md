---
type: prompt
name: gsd:milestone-summary
description: 从里程碑制品生成综合项目摘要，用于团队入职和审查
argument-hint: "[version]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
---

<objective>
生成结构化的里程碑摘要，用于团队入职和项目审查。读取已完成的里程碑制品（ROADMAP、REQUIREMENTS、CONTEXT、SUMMARY、VERIFICATION 文件），生成人类友好的概述，涵盖构建了什么、如何构建以及为什么构建。

目的：使新团队成员能够通过阅读一份文档来理解一个已完成的项目，并可以提出后续问题。
输出：MILESTONE_SUMMARY 写入 `.planning/reports/`，内联展示，可选交互式 Q&A。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/milestone-summary.md
</execution_context>

<context>
**项目文件：**
- `.planning/ROADMAP.md`
- `.planning/PROJECT.md`
- `.planning/STATE.md`
- `.planning/RETROSPECTIVE.md`
- `.planning/milestones/v{version}-ROADMAP.md`（如果已归档）
- `.planning/milestones/v{version}-REQUIREMENTS.md`（如果已归档）
- `.planning/phases/*-*/`（SUMMARY.md、VERIFICATION.md、CONTEXT.md、RESEARCH.md）

**用户输入：**
- 版本：$ARGUMENTS（可选 — 默认为当前/最新里程碑）
</context>

<process>
端到端读取并执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/milestone-summary.md 中的 milestone-summary 工作流。
</process>

<success_criteria>
- 里程碑版本已确定（从参数、STATE.md 或存档扫描）
- 所有可用制品已读取（ROADMAP、REQUIREMENTS、CONTEXT、SUMMARY、VERIFICATION、RESEARCH、RETROSPECTIVE）
- 摘要文档已写入 `.planning/reports/MILESTONE_SUMMARY-v{version}.md`
- 所有 7 个部分已生成（概述、架构、阶段、决策、需求、技术债务、快速开始）
- 摘要已内联展示给用户
- 提供交互式 Q&A
- STATE.md 已更新
</success_criteria>
