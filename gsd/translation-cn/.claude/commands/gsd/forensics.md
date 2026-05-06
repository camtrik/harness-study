---
type: prompt
name: gsd:forensics
description: 对失败的 GSD 工作流进行事后调查 — 诊断问题原因
argument-hint: "[problem description]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
---

<objective>
调查 GSD 工作流执行期间出现的问题。分析 git 历史记录、`.planning/` 制品和文件系统状态，检测异常并生成结构化的诊断报告。

目的：诊断失败或卡住的工作流，以便用户理解根因并采取纠正措施。
输出：取证报告保存到 `.planning/forensics/`，内联展示，可选创建 issue。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/forensics.md
</execution_context>

<context>
**数据来源：**
- `git log`（最近提交、模式、时间间隔）
- `git status` / `git diff`（未提交的工作、冲突）
- `.planning/STATE.md`（当前位置、会话历史）
- `.planning/ROADMAP.md`（阶段范围和进度）
- `.planning/phases/*/`（PLAN.md、SUMMARY.md、VERIFICATION.md、CONTEXT.md）
- `.planning/reports/SESSION_REPORT.md`（上次会话结果）

**用户输入：**
- 问题描述：$ARGUMENTS（可选 — 如未提供将询问）
</context>

<process>
端到端读取并执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/forensics.md 中的 forensics 工作流。
</process>

<success_criteria>
- 从所有可用数据来源收集证据
- 至少检查 4 种异常类型（卡住循环、缺失制品、放弃工作、崩溃/中断）
- 结构化取证报告写入 `.planning/forensics/report-{timestamp}.md`
- 报告内联展示，包含发现、异常和建议
- 提供交互式调查以进行更深入分析
- 如果存在可执行的发现，提供创建 GitHub issue 的选项
</success_criteria>

<critical_rules>
- **只读调查：** 取证期间不要修改项目源文件。仅写入取证报告和更新 STATE.md 会话跟踪。
- **脱敏敏感数据：** 从报告和 issue 中去除绝对路径、API 密钥、token。
- **以证据为基础：** 每个异常必须引用具体的提交、文件或状态数据。
- **无证据不猜测：** 如果数据不足，如实说明 — 不要捏造根因。
</critical_rules>
