---
name: gsd:secure-phase
description: 回溯性验证已完成阶段的威胁缓解措施
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---
<objective>
验证已完成阶段的威胁缓解措施。三种状态：
- (A) SECURITY.md 存在 — 审计并验证缓解措施
- (B) 无 SECURITY.md，但 PLAN.md 中存在威胁模型 — 从制品中运行
- (C) 阶段尚未执行 — 退出并给出指引

输出：更新后的 SECURITY.md。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/secure-phase.md
</execution_context>

<context>
阶段：$ARGUMENTS — 可选，默认为最后完成的阶段。
</context>

<process>
执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/secure-phase.md。
保留所有工作流关卡。
</process>
