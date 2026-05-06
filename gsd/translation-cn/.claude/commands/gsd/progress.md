---
name: gsd:progress
description: 检查进度、推进工作流或分派自由形式意图 — 统一的 GSD 情景命令
argument-hint: "[--forensic | --next | --do \"task description\"]"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
  - AskUserQuestion
---
<objective>
检查项目进度，总结最近的工作和接下来的工作，然后智能地路由到下一步操作。

三种模式：
- **default**：显示进度报告 + 智能路由到下一步操作（执行或规划）。在继续工作前提供情景认知。
- **--next**：自动推进到下一个逻辑步骤，无需手动选择路由。读取 STATE.md、ROADMAP.md 和阶段目录。支持 `--force` 以绕过安全关卡。
- **--do "task description"**：分析自由形式的自然语言并将其分派到最合适的 GSD 命令。从不自行执行工作 — 匹配意图、确认后交给目标命令。
- **--forensic**：在标准进度报告后追加 6 检查完整性审计。
</objective>

<flags>
- **--next**：检测当前项目状态并自动调用下一个逻辑 GSD 工作流步骤。路由前扫描所有前置阶段中未完成的工作。`--next --force` 绕过安全关卡。
- **--do "..."**：智能分派器 — 使用路由规则将自由形式意图匹配到最佳 GSD 命令，确认匹配后将工作交给目标命令。
- **--forensic**：标准进度报告后运行 6 检查完整性审计。
- **（无标志）**：标准进度检查 + 智能路由（路由 A 到 F）。
</flags>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/progress.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/next.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/do.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>
解析 $ARGUMENTS 的第一个 token：
- 如果是 `--next`：去除该标志，执行 next 工作流（传递剩余参数，例如 --force）。
- 如果是 `--do`：去除该标志，将剩余部分作为自由形式意图传递给 do 工作流。
- 否则：端到端执行 progress 工作流（如有 --forensic 则传递）。

保留目标工作流中的所有路由逻辑。
</process>
