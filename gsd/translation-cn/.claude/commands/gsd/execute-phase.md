---
name: gsd:execute-phase
description: 使用基于 wave 的并行化执行阶段中的所有计划
argument-hint: "<phase-number> [--wave N] [--gaps-only] [--interactive] [--tdd]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---
<objective>
使用基于 wave 的并行执行方式执行阶段中的所有计划。

编排器保持精简：发现计划、分析依赖关系、分组为 wave、启动 subagent、收集结果。每个 subagent 加载完整的 execute-plan 上下文并处理自己的计划。

可选 wave 过滤器：
- `--wave N` 仅执行 Wave `N`，用于节奏控制、配额管理或分阶段发布
- 完成所选 wave 后，仅当没有未完成的计划剩余时才会进行阶段验证/完成

标志处理规则：
- 下列可选标志是可用行为，而非隐含的活跃行为
- 仅当其字面 token 出现在 `$ARGUMENTS` 中时，标志才处于活跃状态
- 如果文档中列出的标志未出现在 `$ARGUMENTS` 中，则视为不活跃

上下文预算：约 15% 编排器，每个 subagent 100% 全新。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/execute-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。它们是等效的 — `vscode_askquestions` 是 VS Code Copilot 对同一交互式问题 API 的实现。
</runtime_note>

<context>
阶段：$ARGUMENTS

**可用的可选标志（仅文档说明 — 不会自动激活）：**
- `--wave N` — 仅执行阶段中的 Wave `N`。当你想控制执行节奏或保持在用量限制内时使用。
- `--gaps-only` — 仅执行差距关闭计划（frontmatter 中 `gap_closure: true` 的计划）。在 verify-work 创建修复计划后使用。
- `--interactive` — 顺序内联执行计划（不启动 subagent），任务之间有用户检查点。更低的 token 用量，结对编程风格。最适合小阶段、bug 修复和验证差距。

**活跃标志必须从 `$ARGUMENTS` 推导：**
- 仅当字面 `--wave` token 存在于 `$ARGUMENTS` 中时，`--wave N` 才活跃
- 仅当字面 `--gaps-only` token 存在于 `$ARGUMENTS` 中时，`--gaps-only` 才活跃
- 仅当字面 `--interactive` token 存在于 `$ARGUMENTS` 中时，`--interactive` 才活跃
- 如果以上 token 均未出现，则运行标准全阶段执行流程，不进行特定标志过滤
- 不要仅因为标志在此 prompt 中记录就推断其处于活跃状态

上下文文件在工作流中通过 `gsd-sdk query init.execute-phase` 和每个 subagent 的 `<files_to_read>` 块解析。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/execute-phase.md 中的 execute-phase 工作流。
保留所有工作流关卡（wave 执行、检查点处理、验证、状态更新、路由分发）。
</process>
