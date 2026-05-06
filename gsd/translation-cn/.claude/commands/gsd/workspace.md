---
name: gsd:workspace
description: 管理 GSD 工作区 — 创建、列出或删除隔离的工作区环境
argument-hint: "[--new | --list | --remove] [name]"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
使用单个统一命令管理 GSD 工作区。

模式路由：
- **--new**：创建带有仓库副本和独立 .planning/ 的隔离工作区 → new-workspace 工作流
- **--list**：列出活跃的 GSD 工作区及其状态 → list-workspaces 工作流
- **--remove**：删除 GSD 工作区并清理 worktrees → remove-workspace 工作流
</objective>

<routing>

| 标志 | 操作 | 工作流 |
|------|--------|----------|
| --new | 使用 worktree/clone 策略创建工作区 | new-workspace |
| --list | 扫描 ~/gsd-workspaces/，显示摘要表格 | list-workspaces |
| --remove | 确认并删除工作区目录 | remove-workspace |

</routing>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/new-workspace.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/list-workspaces.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/remove-workspace.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
参数：$ARGUMENTS

解析 $ARGUMENTS 的第一个 token：
- 如果是 `--new`：去除该标志，将剩余部分（--name、--repos、--path、--strategy、--branch、--auto 标志）传递给 new-workspace 工作流
- 如果是 `--list`：执行 list-workspaces 工作流（无需参数）
- 如果是 `--remove`：去除该标志，将剩余部分（workspace-name）传递给 remove-workspace 工作流
- 否则（无标志）：显示用法 — 需要 --new、--list 或 --remove 之一
</context>

<process>
1. 从 $ARGUMENTS 中解析前置标志。
2. 根据上述路由表，加载并端到端执行相应的工作流。
3. 保留目标工作流的所有工作流关卡（验证、审批、提交、路由分发）。
</process>
