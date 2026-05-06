---
name: gsd:health
description: 诊断规划目录健康状况并可选择性地修复问题
argument-hint: "[--repair] [--context]"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
验证 `.planning/` 目录的完整性并报告可操作的问题。检查缺失文件、无效配置、不一致状态和孤立计划。

`--context` 运行一个正交检查：正在运行会话的上下文利用率。工作流要求模型提供 tokensUsed + contextWindow，调用 `gsd-sdk query validate.context`，并渲染三种状态之一：

| 利用率 | 状态    | 操作                                                |
|-------------|----------|-------------------------------------------------------|
| < 60%       | healthy  | 无需操作 — 上下文充裕                    |
| 60% – 70%   | warning  | 建议 `/gsd-thread` 以全新开始                |
| ≥ 70%       | critical | 推理质量可能在断裂点之后下降 |
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/health.md
</execution_context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/health.md 中的 health 工作流。
从参数解析 `--repair` 和 `--context` 标志并传递给工作流。
</process>
