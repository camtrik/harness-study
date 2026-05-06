---
name: gsd:fast
description: 内联执行简单任务 — 无需 subagent，无规划开销
argument-hint: "[task description]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

<objective>
在当前上下文中直接执行简单任务，无需启动 subagent
或生成 PLAN.md 文件。用于太小不值得规划开销的任务：
打字错误修复、配置更改、小型重构、遗漏提交、简单添加。

这不是 /gsd-quick 的替代品 — 对于需要研究、
多步骤规划或验证的任务，请使用 /gsd-quick。/gsd-fast 适用于
你能用一句话描述且两分钟内执行完成的任务。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/fast.md
</execution_context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/fast.md 中的 fast 工作流。
</process>
