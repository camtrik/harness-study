---
name: gsd:autonomous
description: 自主运行所有剩余阶段 — 每个阶段依次执行 discuss→plan→execute
argument-hint: "[--from N] [--to N] [--only N] [--interactive]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
  - Agent
---
<objective>
自主执行里程碑中所有剩余阶段。每个阶段执行：discuss → plan → execute。仅在需要用户决策时暂停（灰区接受、阻塞项、验证请求）。

使用 ROADMAP.md 进行阶段发现，并通过 Skill() 扁平化调用来执行每个阶段命令。所有阶段完成后：里程碑审计 → 完成 → 清理。

**创建/更新：**
- `.planning/STATE.md` — 每个阶段完成后更新
- `.planning/ROADMAP.md` — 每个阶段完成后更新进度
- 阶段制品 — 每个阶段的 CONTEXT.md、PLANs、SUMMARYs

**之后：** 里程碑已完成并清理。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/autonomous.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
可选标志：
- `--from N` — 从阶段 N 开始，而非第一个未完成的阶段。
- `--to N` — 在阶段 N 完成后停止（停止而不推进到下一个阶段）。
- `--only N` — 仅执行阶段 N（单阶段模式）。
- `--interactive` — 内联运行 discuss 并支持提问（非自动回答），然后将 plan→execute 调度为后台 agent。保持主上下文精简，同时保留用户在决策上的输入。

项目上下文、阶段列表和状态在工作流中通过 init 命令（`gsd-sdk query init.milestone-op`、`gsd-sdk query roadmap.analyze`）解析。无需前置上下文加载。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/autonomous.md 中的 autonomous 工作流。
保留所有工作流关卡（阶段发现、逐阶段执行、阻塞处理、进度显示）。
</process>
