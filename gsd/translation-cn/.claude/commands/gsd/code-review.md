---
name: gsd:code-review
description: 审查某个阶段修改的源文件，检查 bug、安全问题和代码质量问题
argument-hint: "<phase-number> [--depth=quick|standard|deep] [--files file1,file2,...] [--fix [--all] [--auto]]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---
<objective>
审查某个阶段修改的源文件，检查 bug、安全漏洞和代码质量问题。

启动 gsd-code-reviewer agent，按照指定的深度级别分析代码。在阶段目录中生成 REVIEW.md 制品，包含按严重级别分类的发现项。

参数：
- 阶段编号（必填）— 要审查哪个阶段的修改（例如 "2" 或 "02"）
- `--depth=quick|standard|deep`（可选）— 审查深度级别，覆盖 workflow.code_review_depth 配置
  - quick：仅模式匹配（约 2 分钟）
  - standard：逐文件分析及语言特定检查（约 5-15 分钟，默认值）
  - deep：跨文件分析，包括导入图和调用链（约 15-30 分钟）
- `--files file1,file2,...`（可选）— 以逗号分隔的显式文件列表，跳过 SUMMARY/git 范围确定（范围确定的最高优先级）
- `--fix`（可选）— 审查完成后（或 REVIEW.md 已存在时），自动应用找到的修复。启动 gsd-code-fixer agent。接受子标志：
  - `--all` — 修复范围包含 Info 级别的发现项（默认：仅 Critical + Warning）
  - `--auto` — 启用修复 + 重新审查迭代循环，最多 3 次迭代

输出：阶段目录中的 {padded_phase}-REVIEW.md + 发现项的内联摘要
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/code-review.md
</execution_context>

<context>
阶段：$ARGUMENTS（第一个位置参数为阶段编号）

从 $ARGUMENTS 解析的可选标志：
- `--depth=VALUE` — 深度覆盖（quick|standard|deep）。如果提供，覆盖 workflow.code_review_depth 配置。
- `--files=file1,file2,...` — 显式文件列表覆盖。根据 D-08，对文件范围确定具有最高优先级。提供时，工作流完全跳过 SUMMARY.md 提取和 git diff 回退。

上下文文件（CLAUDE.md、SUMMARY.md、阶段状态）在工作流中通过 `gsd-sdk query init.phase-op` 解析，并通过 `<files_to_read>` 块委托给 agent。
</context>

<process>
此命令是一个薄调度层。它解析参数并委托给工作流。

端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/code-review.md 中的 code-review 工作流。

工作流（而非此命令）执行以下关卡：
- 阶段验证（在配置关卡之前）
- 配置关卡检查（workflow.code_review）
- 文件范围确定（--files 覆盖 > SUMMARY.md > git diff 回退）
- 空范围检查（无文件时跳过）
- Agent 启动（gsd-code-reviewer）
- 结果展示（内联摘要 + 后续步骤）
</process>
