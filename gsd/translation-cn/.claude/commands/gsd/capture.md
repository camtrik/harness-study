---
name: gsd:capture
description: 将想法、任务、笔记和种子捕获到其目标位置
argument-hint: "[--note | --backlog | --seed | --list] [text]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
将想法、任务、笔记和种子捕获到 GSD 系统中的适当目标位置。

模式路由：
- **default**（无标志）：作为结构化待办事项捕获，供后续处理 → add-todo 工作流
- **--note**：零摩擦想法捕获（追加/列出/提升）→ note 工作流
- **--backlog**：将想法添加到 backlog 暂存区（999.x 编号）→ add-backlog 工作流
- **--seed**：捕获带有触发条件的前瞻性想法 → plant-seed 工作流
- **--list**：列出待处理待办事项并选择一个来处理 → check-todos 工作流
</objective>

<routing>

| 标志 | 目标位置 | 工作流 |
|------|-------------|----------|
| (无) | .planning/todos/ 中的结构化待办事项 | add-todo |
| --note | 带时间戳的笔记文件、列表或提升 | note |
| --backlog | ROADMAP.md backlog 部分（999.x） | add-backlog |
| --seed | .planning/seeds/SEED-NNN-slug.md | plant-seed |
| --list | 交互式待办事项浏览器 + 操作路由器 | check-todos |

</routing>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/add-todo.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/note.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/add-backlog.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/plant-seed.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/check-todos.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
参数：$ARGUMENTS

解析 $ARGUMENTS 的第一个 token：
- 如果是 `--note`：去除该标志，将剩余部分传递给 note 工作流
- 如果是 `--backlog`：去除该标志，将剩余部分传递给 add-backlog 工作流
- 如果是 `--seed`：去除该标志，将剩余部分传递给 plant-seed 工作流
- 如果是 `--list`：将剩余部分（可选的区域过滤器）传递给 check-todos 工作流
- 否则：将整个 $ARGUMENTS 传递给 add-todo 工作流
</context>

<process>
1. 从 $ARGUMENTS 中解析前置标志（如果有）。
2. 根据上述路由表，加载并端到端执行相应的工作流。
3. 保留目标工作流的所有工作流关卡（目录结构、重复检测、提交等）。
</process>
