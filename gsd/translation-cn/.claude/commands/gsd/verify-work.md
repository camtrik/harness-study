---
name: gsd:verify-work
description: 通过对话式 UAT 验证已构建的功能
argument-hint: "[phase number, e.g., '4']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Edit
  - Write
  - Task
---
<objective>
通过具有持久化状态的对话式测试验证已构建的功能。

目的：从用户视角确认 Claude 构建的内容确实有效。一次一个测试，纯文本回答，没有审问感。发现问题时，自动诊断、制定修复计划并准备执行。

输出：{phase_num}-UAT.md 跟踪所有测试结果。如果发现问题：已诊断的差距、已验证的修复计划，可供 /gsd-execute-phase 使用
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/verify-work.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/UAT.md
</execution_context>

<context>
阶段：$ARGUMENTS（可选）
- 如果提供：测试特定阶段（例如 "4"）
- 如果未提供：检查活跃会话或提示输入阶段

上下文文件在工作流中解析（`init verify-work`），并通过 `<files_to_read>` 块委托。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/verify-work.md 中的 verify-work 工作流。
保留所有工作流关卡（会话管理、测试展示、诊断、修复计划制定、路由分发）。
</process>
