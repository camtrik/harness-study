---
name: gsd:profile-user
description: 生成开发者行为画像并创建 Claude 可发现的制品
argument-hint: "[--questionnaire] [--refresh]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
---

<objective>
通过会话分析（或问卷）生成开发者行为画像，并生成制品（USER-PROFILE.md、/gsd-dev-preferences、CLAUDE.md 部分），个性化 Claude 的响应。

路由到 profile-user 工作流，编排完整流程：同意关卡、会话分析或问卷回退、画像生成、结果显示和制品选择。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/profile-user.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS 中的标志：
- `--questionnaire` — 完全跳过多会话分析，仅使用问卷路径
- `--refresh` — 即使已有画像也重新构建，备份旧画像，显示维度差异
</context>

<process>
端到端执行 profile-user 工作流。

工作流处理所有逻辑，包括：
1. 初始化和现有画像检测
2. 会话分析前的同意关卡
3. 会话扫描和数据充分性检查
4. 会话分析（profiler agent）或问卷回退
5. 跨项目分歧解决
6. 画像写入 USER-PROFILE.md
7. 带报告卡片和亮点的结果显示
8. 制品选择（dev-preferences、CLAUDE.md 部分）
9. 顺序制品生成
10. 带刷新差异的摘要（如适用）
</process>
