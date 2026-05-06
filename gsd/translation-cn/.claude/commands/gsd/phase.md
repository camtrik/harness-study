---
name: gsd:phase
description: 在 ROADMAP.md 中对阶段进行 CRUD — 添加、插入、删除或编辑阶段
argument-hint: "[--insert | --remove | --edit] <phase-name-or-number>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

<objective>
使用单个统一命令管理 ROADMAP.md 中的阶段。

模式路由：
- **default**（无标志）：在当前里程碑末尾添加一个新的整数阶段 → add-phase 工作流
- **--insert**：在现有阶段之间以小数阶段（例如 72.1）的形式插入紧急工作 → insert-phase 工作流
- **--remove**：删除未来阶段并重新编号后续阶段 → remove-phase 工作流
- **--edit**：就地编辑现有阶段的任何字段 → edit-phase 工作流
</objective>

<routing>

| 标志 | 操作 | 工作流 |
|------|--------|----------|
| (无) | 在里程碑末尾添加新的整数阶段 | add-phase |
| --insert | 在指定阶段之后插入小数阶段（例如 72.1） | insert-phase |
| --remove | 删除未来阶段，重新编号后续阶段 | remove-phase |
| --edit | 就地编辑现有阶段的字段 | edit-phase |

</routing>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/add-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/insert-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/remove-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/edit-phase.md
</execution_context>

<context>
参数：$ARGUMENTS

解析 $ARGUMENTS 的第一个 token：
- 如果是 `--insert`：去除该标志，将剩余部分（格式：<after-phase-number> <description>）传递给 insert-phase 工作流
- 如果是 `--remove`：去除该标志，将剩余部分（阶段编号）传递给 remove-phase 工作流
- 如果是 `--edit`：去除该标志，将剩余部分（phase-number [--force]）传递给 edit-phase 工作流
- 否则：将整个 $ARGUMENTS（阶段描述）传递给 add-phase 工作流

路线图和状态在工作流中通过 `init phase-op` 和针对性读取解析。
</context>

<process>
1. 从 $ARGUMENTS 中解析前置标志（如果有）。
2. 根据上述路由表，加载并端到端执行相应的工作流。
3. 保留目标工作流的所有验证关卡。
</process>
