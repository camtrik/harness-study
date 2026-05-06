---
name: gsd:spike
description: 通过体验式探索对想法进行 spike，或建议接下来 spike 什么（前沿模式）
argument-hint: "[idea to validate] [--quick] [--text] [--wrap-up] or [frontier]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
  - WebFetch
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---
<objective>
通过体验式探索对想法进行 spike — 构建聚焦实验来感受未来应用的各个部分，
验证可行性，并为实际构建产生经过验证的知识。
Spike 文件位于 `.planning/spikes/` 中，并与 GSD 提交模式、状态跟踪
和交接工作流集成。

两种模式：
- **Idea 模式**（默认）— 描述一个要 spike 的想法
- **Frontier 模式**（无参数或 "frontier"）— 分析现有的 spike 全景图，并提出集成和前沿方面的 spike 建议

不需要 `/gsd-new-project` — 需要时自动创建 `.planning/spikes/`。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spike.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spike-wrap-up.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。
</runtime_note>

<context>
想法：$ARGUMENTS

**可用标志：**
- `--quick` — 跳过分解/对齐，直接进入构建。当已经知道要 spike 什么时使用。
- `--text` — 使用纯文本编号列表而非 AskUserQuestion（用于非 Claude 运行时）。
- `--wrap-up` — 将 spike 发现打包为持久化项目 skill，供未来构建对话使用。运行 spike-wrap-up 工作流。
</context>

<process>
解析 $ARGUMENTS 的第一个 token：
- 如果是 `--wrap-up`：去除该标志，执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spike-wrap-up.md 中的 spike-wrap-up 工作流。
- 否则：将整个 $ARGUMENTS 作为想法传递给 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spike.md 中的 spike 工作流，端到端执行。

保留所有工作流关卡（前置 spike 检查、分解、研究、风险排序、可观测性评估、验证、MANIFEST 更新、提交模式）。
</process>
