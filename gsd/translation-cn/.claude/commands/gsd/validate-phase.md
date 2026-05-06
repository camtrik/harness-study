---
name: gsd:validate-phase
description: 回溯性审计并填补已完成阶段的 Nyquist 验证差距
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
审计已完成阶段的 Nyquist 验证覆盖率。三种状态：
- (A) VALIDATION.md 存在 — 审计并填补差距
- (B) 无 VALIDATION.md，SUMMARY.md 存在 — 从制品重建
- (C) 阶段尚未执行 — 退出并给出指引

输出：更新后的 VALIDATION.md + 生成的测试文件。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/validate-phase.md
</execution_context>

<context>
阶段：$ARGUMENTS — 可选，默认为最后完成的阶段。
</context>

<process>
执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/validate-phase.md。
保留所有工作流关卡。
</process>
