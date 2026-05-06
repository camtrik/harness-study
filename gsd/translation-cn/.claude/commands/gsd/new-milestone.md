---
name: gsd:new-milestone
description: 启动新的里程碑循环 — 更新 PROJECT.md 并路由到需求阶段
argument-hint: "[milestone name, e.g., 'v1.1 Notifications']"
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
启动新里程碑：提问 → 研究（可选）→ 需求 → 路线图。

存量项目的 new-project 等价操作。项目已存在，PROJECT.md 有历史记录。收集"下一步做什么"，更新 PROJECT.md，然后运行需求 → 路线图循环。

**创建/更新：**
- `.planning/PROJECT.md` — 更新新里程碑目标
- `.planning/research/` — 领域研究（可选，仅限 NEW 特性）
- `.planning/REQUIREMENTS.md` — 此里程碑的范围化需求
- `.planning/ROADMAP.md` — 阶段结构（继续编号）
- `.planning/STATE.md` — 为新的里程碑重置

**之后：** `/gsd-plan-phase [N]` 开始执行。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/new-milestone.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/questioning.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/project.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/requirements.md
</execution_context>

<context>
里程碑名称：$ARGUMENTS（可选 - 如果未提供将提示）

项目和里程碑上下文文件在工作流中解析（`init new-milestone`），并在使用 subagent 的地方通过 `<files_to_read>` 块委托。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/new-milestone.md 中的 new-milestone 工作流。
保留所有工作流关卡（验证、提问、研究、需求、路线图审批、提交）。
</process>
