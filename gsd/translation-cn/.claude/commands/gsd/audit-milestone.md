---
name: gsd:audit-milestone
description: 在归档前对照原始意图审计里程碑完成情况
argument-hint: "[version]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - Write
---
<objective>
验证里程碑是否达到了其完成定义。检查需求覆盖率、跨阶段集成以及端到端流程。

**此命令本身就是编排器。** 读取现有的 VERIFICATION.md 文件（各阶段在 execute-phase 期间已验证），汇总技术债务和延期差距，然后启动集成检查器来检查跨阶段连接。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/audit-milestone.md
</execution_context>

<context>
版本：$ARGUMENTS（可选 — 默认为当前里程碑）

核心规划文件在工作流中解析（`init milestone-op`），仅按需加载。

**已完成的工作：**
Glob: .planning/phases/*/*-SUMMARY.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/audit-milestone.md 中的 audit-milestone 工作流。
保留所有工作流关卡（范围确定、验证读取、集成检查、需求覆盖率、路由分发）。
</process>
