---
name: gsd:ultraplan-phase
description: "[BETA] 将 plan phase 卸载到 Claude Code 的 ultraplan 云；在浏览器中审查并导入回来"
argument-hint: "[phase-number]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
将 GSD 的 plan phase 卸载到 Claude Code 的 ultraplan 云基础设施。

Ultraplan 在远程云会话中起草计划，而你的终端保持空闲。
在浏览器中审查和评论计划，然后通过 /gsd-import --from 将其导入回来。

⚠ 测试版：ultraplan 处于研究预览阶段。使用 /gsd-plan-phase 进行稳定的本地规划。
要求：Claude Code v2.1.91+、claude.ai 账户、GitHub 仓库。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ultraplan-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
端到端执行 ultraplan-phase 工作流。
</process>
