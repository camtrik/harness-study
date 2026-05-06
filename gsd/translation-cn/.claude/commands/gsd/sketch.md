---
name: gsd:sketch
description: 使用一次性 HTML 原型探索 UI/设计想法，或建议接下来 sketch 什么（前沿模式）
argument-hint: "[design idea to explore] [--quick] [--text] [--wrap-up] or [frontier]"
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
在承诺实现之前，通过一次性 HTML 原型探索设计方向。
每次 sketch 生成 2-3 个变体进行比较。Sketch 文件位于 `.planning/sketches/` 中，
并与 GSD 提交模式、状态跟踪和交接工作流集成。加载 spike
结果以在真实数据结构和已验证的交互模式中构建原型。

两种模式：
- **Idea 模式**（默认）— 描述要 sketch 的设计想法
- **Frontier 模式**（无参数或 "frontier"）— 分析现有的 sketch 全景图，并提出一致性和前沿方面的 sketch 建议

不需要 `/gsd-new-project` — 需要时自动创建 `.planning/sketches/`。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/sketch.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/sketch-wrap-up.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/sketch-theme-system.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/sketch-interactivity.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/sketch-tooling.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/sketch-variant-patterns.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。
</runtime_note>

<context>
设计想法：$ARGUMENTS

**可用标志：**
- `--quick` — 跳过风格/方向收集，直接进入分解和构建。当设计方向已明确时使用。
- `--wrap-up` — 将 sketch 设计发现打包为持久化项目 skill，供未来构建对话使用。运行 sketch-wrap-up 工作流。
</context>

<process>
解析 $ARGUMENTS 的第一个 token：
- 如果是 `--wrap-up`：去除该标志，端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/sketch-wrap-up.md 中的 sketch-wrap-up 工作流。
- 否则：端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/sketch.md 中的 sketch 工作流。

保留所有工作流关卡（收集、分解、目标技术栈研究、变体评估、MANIFEST 更新、提交模式）。
</process>
