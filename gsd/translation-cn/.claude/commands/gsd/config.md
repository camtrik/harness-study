---
name: gsd:config
description: 配置 GSD 设置 — 工作流开关、高级选项、集成和模型配置文件
argument-hint: "[--advanced | --integrations | --profile <name>]"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
通过单个统一命令交互式配置 GSD 设置。

模式路由：
- **default**（无标志）：常用开关项（model、research、plan_check、verifier、branching）→ settings 工作流
- **--advanced**：高级用户选项（规划调优、超时、分支模板、跨 AI 执行）→ settings-advanced 工作流
- **--integrations**：第三方 API 密钥、code-review CLI 路由、agent-skill 注入 → settings-integrations 工作流
- **--profile <name>**：切换模型配置文件（quality|balanced|budget|inherit）→ set-profile（内联）
</objective>

<routing>

| 标志 | 操作 | 工作流 |
|------|--------|----------|
| (无) | 交互式 5 问题通用配置提示 | settings |
| --advanced | 高级用户选项：规划、执行、讨论、跨 AI、git、运行时 | settings-advanced |
| --integrations | API 密钥（Brave/Firecrawl/Exa）、审查 CLI 路由、agent skills | settings-integrations |
| --profile &lt;name&gt; | 无需交互提示直接切换模型配置文件 | gsd-sdk config-set-model-profile |

</routing>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/settings.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/settings-advanced.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/settings-integrations.md
</execution_context>

<context>
参数：$ARGUMENTS

解析 $ARGUMENTS 的第一个 token：
- 如果是 `--advanced`：去除该标志，执行 settings-advanced 工作流
- 如果是 `--integrations`：去除该标志，执行 settings-integrations 工作流
- 如果以 `--profile` 开头：提取配置文件名称（`--profile` 之后的部分），然后：
  1. **预检（#2439）：** 通过 `command -v gsd-sdk` 验证 `gsd-sdk` 在 PATH 中。
     如果不存在，发出安装提示 `Install GSD via 'npm i -g get-shit-done'` 并停止 —
     不要直接调用 `gsd-sdk`（避免晦涩的 `command not found: gsd-sdk` 失败）。
  2. 运行：`gsd-sdk query config-set-model-profile <profile-name> --raw` 并逐字显示输出。
- 否则：执行 settings 工作流（无需参数）
</context>

<process>
1. 从 $ARGUMENTS 中解析前置标志（如果有）。
2. 加载并端到端执行相应的工作流，或为 --profile 运行内联 SDK 命令。
3. 保留目标工作流的所有工作流关卡。
</process>
