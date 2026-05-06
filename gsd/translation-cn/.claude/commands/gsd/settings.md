---
name: gsd:settings
description: 配置 GSD 工作流开关和模型配置文件
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
通过多问题提示交互式配置 GSD 工作流 agent 和模型配置文件。

路由到 settings 工作流，该工作流处理：
- 确保配置存在
- 当前设置读取和解析
- 交互式 5 问题提示（model、research、plan_check、verifier、branching）
- 配置合并和写入
- 带快速命令参考的确认显示
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/settings.md
</execution_context>

<process>
**遵循 settings 工作流**，来自 `@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/settings.md`。

工作流处理所有逻辑，包括：
1. 缺失时使用默认值创建配置文件
2. 当前配置读取
3. 带预选值的交互式设置展示
4. 答案解析和配置合并
5. 文件写入
6. 确认显示
</process>
