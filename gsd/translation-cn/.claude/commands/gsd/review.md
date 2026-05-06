---
name: gsd:review
description: 请求外部 AI CLI 对阶段计划进行跨 AI 同行审查
argument-hint: "--phase N [--gemini] [--claude] [--codex] [--opencode] [--qwen] [--cursor] [--all]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
---

<objective>
调用外部 AI CLI（Gemini、Claude、Codex、OpenCode、Qwen Code、Cursor）独立审查阶段计划。
生成结构化的 REVIEWS.md，包含每个审查者的反馈，可以通过
/gsd-plan-phase --reviews 反馈回规划流程。

**流程：** 检测 CLI → 构建审查 prompt → 调用每个 CLI → 收集响应 → 写入 REVIEWS.md
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/review.md
</execution_context>

<context>
阶段编号：从 $ARGUMENTS 提取（必填）

**标志：**
- `--gemini` — 包含 Gemini CLI 审查
- `--claude` — 包含 Claude CLI 审查（使用单独的会话）
- `--codex` — 包含 Codex CLI 审查
- `--opencode` — 包含 OpenCode 审查（使用用户 OpenCode 配置中的模型）
- `--qwen` — 包含 Qwen Code 审查（阿里巴巴 Qwen 模型）
- `--cursor` — 包含 Cursor agent 审查
- `--all` — 包含所有可用的 CLI
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/review.md 中的 review 工作流。
</process>
