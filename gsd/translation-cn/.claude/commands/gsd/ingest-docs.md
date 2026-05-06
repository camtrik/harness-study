---
name: gsd:ingest-docs
description: 从仓库中现有的 ADR、PRD、SPEC 和文档引导或合并 .planning/ 设置
argument-hint: "[path] [--mode new|merge] [--manifest <file>] [--resolve auto|interactive]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
---

<objective>
一次性从多个现有的规划文档 — ADR、PRD、SPEC、DOC — 构建完整的 `.planning/` 设置（或合并到现有设置）。

- **全新引导**（`--mode new`，当 `.planning/` 不存在时为默认）：根据合成的文档内容生成 PROJECT.md + REQUIREMENTS.md + ROADMAP.md + STATE.md，将最终生成委托给 `gsd-roadmapper`。
- **合并到现有设置**（`--mode merge`，当 `.planning/` 存在时为默认）：追加从导入文档中推导出的阶段和需求；与现有锁定决策的任何矛盾之处将硬阻塞。

使用优先级规则 `ADR > SPEC > PRD > DOC`（可通过 manifest 覆盖）自动合成大多数冲突。在 `.planning/INGEST-CONFLICTS.md` 中呈现未解决的案例，分为三个桶：auto-resolved、competing-variants、unresolved-blockers。共享冲突引擎的 BLOCKER 关卡可防止在存在未解决矛盾时写入任何目标文件。

**输入：** 目录约定发现（`docs/adr/`、`docs/prd/`、`docs/specs/`、`docs/rfc/`，根级别 `{ADR,PRD,SPEC,RFC}-*.md`），或显式的 `--manifest <file>` YAML，其中列出每个文档的 `{path, type, precedence?}`。

**v1 限制：** 每次调用最多 50 个文档；`--resolve interactive` 保留给未来版本。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/ingest-docs.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/gate-prompts.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/doc-conflict-engine.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
端到端执行 ingest-docs 工作流。保留所有审批关卡（发现、冲突报告、路由分发）和 BLOCKER 安全规则。
</process>
