---
name: gsd:plan-review-convergence
description: "跨 AI 计划审查收敛循环 — 反复整合审查反馈重新规划，直到不存在 HIGH 级别的问题"
argument-hint: "<phase> [--codex] [--gemini] [--claude] [--opencode] [--ollama] [--lm-studio] [--llama-cpp] [--text] [--ws <name>] [--all] [--max-cycles N]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
---

<objective>
跨 AI 计划审查收敛循环 — 围绕 gsd-review 和 gsd-planner 的外部修订关卡。
重复进行：使用外部 AI CLI 审查计划 → 如果发现 HIGH 级别的问题 → 使用 --reviews 反馈重新规划 → 重新审查。当没有 HIGH 级别问题剩余或达到最大循环次数时停止。

**流程：** Agent→Skill("gsd-plan-phase") → Agent→Skill("gsd-review") → 检查 HIGHs → Agent→Skill("gsd-plan-phase --reviews") → Agent→Skill("gsd-review") → …… → 收敛或升级

用外部 AI 审查者（codex、gemini 等）替换 gsd-plan-phase 内部的 gsd-plan-checker。每一步在调用现有对应 Skill 的隔离 Agent 中运行 — 编排器仅做循环控制。

**编排器角色：** 解析参数，验证阶段，为现有 Skill 启动 Agent，检查 HIGHs，停滞检测，升级关卡。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/plan-review-convergence.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/revision-loop.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/gates.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/agent-contracts.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。它们是等效的 — `vscode_askquestions` 是 VS Code Copilot 对同一交互式问题 API 的实现。不要因为 `AskUserQuestion` 看起来不可用就跳过提问步骤；改用 `vscode_askquestions`。
</runtime_note>

<context>
阶段编号：从 $ARGUMENTS 提取（必填）

**标志：**
- `--codex` — 使用 Codex CLI 作为审查者（如果未指定审查者则为默认）
- `--gemini` — 使用 Gemini CLI 作为审查者
- `--claude` — 使用 Claude CLI 作为审查者（单独的会话）
- `--opencode` — 使用 OpenCode 作为审查者
- `--ollama` — 使用本地 Ollama 服务器作为审查者（兼容 OpenAI，默认主机 `http://localhost:11434`；通过 `review.models.ollama` 配置模型）
- `--lm-studio` — 使用本地 LM Studio 服务器作为审查者（兼容 OpenAI，默认主机 `http://localhost:1234`；通过 `review.models.lm_studio` 配置模型）
- `--llama-cpp` — 使用本地 llama.cpp 服务器作为审查者（兼容 OpenAI，默认主机 `http://localhost:8080`；通过 `review.models.llama_cpp` 配置模型）
- `--all` — 使用所有可用的 CLI 和正在运行的本地模型服务器
- `--max-cycles N` — 最大重新规划→审查循环次数（默认：3）

**特性开关：** 此命令需要 `workflow.plan_review_convergence=true`。启用方式：
`gsd config-set workflow.plan_review_convergence true`
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/plan-review-convergence.md 中的 plan-review-convergence 工作流。
保留所有工作流关卡（预检、修订循环、停滞检测、升级）。
</process>
