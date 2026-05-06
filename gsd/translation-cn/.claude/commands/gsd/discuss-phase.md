---
name: gsd:discuss-phase
description: 在规划前通过自适应提问收集阶段上下文
argument-hint: "<phase> [--all] [--auto] [--chain] [--batch] [--analyze] [--text] [--power]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---

<objective>
提取下游 agent 所需实现决策 — researcher 和 planner 将使用 CONTEXT.md 来了解需要调查什么以及哪些选择已被锁定。

**工作原理：**
1. 加载前置上下文（PROJECT.md、REQUIREMENTS.md、STATE.md、先前的 CONTEXT.md 文件）
2. 扫描代码库以寻找可复用资产和模式
3. 分析阶段 — 跳过先前阶段已决定的灰区
4. 展示剩余的灰区 — 用户选择要讨论哪些
5. 深入探讨每个选定的区域直到满意
6. 创建 CONTEXT.md，其中的决策足够清晰，下游 agent 无需再次询问用户即可执行

**输出：** `{phase_num}-CONTEXT.md` — 决策清晰到下游 agent 可以无需再次询问用户即可执行的程度
</objective>

<execution_context>
工作流文件在下面 <process> 部分按需加载 — 非前置加载。
在阅读模式路由说明之前，不要预加载任何工作流文件。
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。它们是等效的 — `vscode_askquestions` 是 VS Code Copilot 对同一交互式问题 API 的实现。
</runtime_note>

<context>
阶段编号：$ARGUMENTS（必填）

上下文文件在工作流中通过 `init phase-op` 和 roadmap/state 工具调用解析。
</context>

<process>
**模式路由：**
```bash
DISCUSS_MODE=$(gsd-sdk query config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

如果 `DISCUSS_MODE` 为 `"assumptions"`：
读取并端到端执行 `/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/discuss-phase-assumptions.md`。

如果 `DISCUSS_MODE` 为 `"discuss"`（或未设置，或任何其他值）：
读取并端到端执行 `/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/discuss-phase.md`。

**强制要求：** 在执行任何操作之前读取适当的工作流文件。此命令文件中的 objective 和 success_criteria 部分仅是摘要 — 工作流文件包含完整的逐步流程，包括所有必需的行为、配置检查和交互模式。不要根据摘要即兴发挥。

**延迟加载：** `templates/context.md` 在活跃工作流的 `write_context` 步骤中加载。`discuss-phase-power.md` 在 `discuss-phase.md` 中检测到 `--power` 时加载。均不要在此处加载。
</process>

<success_criteria>
- 前置上下文已加载并应用（不再重复询问已决定的问题）
- 通过智能分析识别出灰区
- 用户选择了要讨论哪些区域
- 每个选定区域均已彻底探讨直至满意
- 范围蔓延已重定向到延迟想法
- CONTEXT.md 捕获的是决策，而非模糊愿景
- 用户知晓后续步骤
</success_criteria>
