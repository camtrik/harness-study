---
name: gsd:docs-update
description: 生成或更新项目文档，并与代码库进行验证核对
argument-hint: "[--force] [--verify-only]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---
<objective>
为当前项目生成并更新最多 9 个文档文件。每个文档类型由 gsd-doc-writer subagent 编写，该 agent 直接探索代码库 — 不会出现虚构路径、虚假端点或过时签名。

标志处理规则：
- 下列可选标志是可用行为，而非隐含的活跃行为
- 仅当其字面 token 出现在 `$ARGUMENTS` 中时，标志才处于活跃状态
- 如果文档中列出的标志未出现在 `$ARGUMENTS` 中，则视为不活跃
- `--force`：跳过保留提示，无论现有内容或 GSD 标记如何，重新生成所有文档
- `--verify-only`：检查现有文档与代码库的一致性，不生成文件（完整验证需要 Phase 4 verifier）
- 如果 `$ARGUMENTS` 中同时出现 `--force` 和 `--verify-only`，`--force` 优先
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/docs-update.md
</execution_context>

<context>
参数：$ARGUMENTS

**可用的可选标志（仅文档说明 — 不会自动激活）：**
- `--force` — 重新生成所有文档。覆盖手写和 GSD 文档。无保留提示。
- `--verify-only` — 检查现有文档与代码库的一致性。不写入任何文件。报告 VERIFY 标记计数。完整的代码库事实核查需要 gsd-doc-verifier agent（Phase 4）。

**活跃标志必须从 `$ARGUMENTS` 推导：**
- 仅当字面 `--force` token 存在于 `$ARGUMENTS` 中时，`--force` 才活跃
- 仅当字面 `--verify-only` token 存在于 `$ARGUMENTS` 中时，`--verify-only` 才活跃
- 如果两个 token 均未出现，则运行标准全阶段生成流程
- 不要仅因为标志在此 prompt 中记录就推断其处于活跃状态
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/docs-update.md 中的 docs-update 工作流。
保留所有工作流关卡（preservation_check、标志处理、wave 执行、monorepo 调度、提交、报告）。
</process>
