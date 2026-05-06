---
name: gsd:manager
description: 交互式命令中心，在一个终端中管理多个阶段
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Skill
  - Task
---
<objective>
用于管理里程碑的单终端命令中心。显示所有阶段的仪表板，带有可视化状态指示器，推荐最优下一步操作，并分派工作 — discuss 内联运行，plan/execute 作为后台 agent 运行。

专为希望在一个终端中跨阶段并行处理工作的高级用户设计：在一个阶段进行 discuss 的同时，另一个阶段在后台进行 plan 或 execute。

**创建/更新：**
- 不直接创建文件 — 通过 Skill() 和后台 Task agent 分派到现有的 GSD 命令。
- 读取 `.planning/STATE.md`、`.planning/ROADMAP.md`、阶段目录以获取状态。

**之后：** 用户完成管理后退出，或所有阶段完成后提示里程碑生命周期操作。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/manager.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
无需参数。需要一个带有 ROADMAP.md 和 STATE.md 的活跃里程碑。

项目上下文、阶段列表、依赖关系和建议在工作流中通过 `gsd-sdk query init.manager` 解析。无需前置上下文加载。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/manager.md 中的 manager 工作流。
保持仪表板刷新循环，直到用户退出或所有阶段完成。
</process>
