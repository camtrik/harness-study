---
name: gsd:add-tests
description: 根据 UAT 标准和实际实现为已完成的阶段生成测试
argument-hint: "<phase> [additional instructions]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
argument-instructions: |
  将参数解析为阶段编号（整数、小数或带字母后缀），外加可选的自由文本说明。
  示例：/gsd-add-tests 12
  示例：/gsd-add-tests 12 重点关注定价模块的边界情况
---
<objective>
为已完成的阶段生成单元测试和 E2E 测试，以其 SUMMARY.md、CONTEXT.md 和 VERIFICATION.md 作为规格说明。

分析实现文件，将其分类为 TDD（单元）、E2E（浏览器）或跳过类别，向用户展示测试计划并请求批准，然后按照 RED-GREEN 规范生成测试。

输出：带提交信息 `test(phase-{N}): add unit and E2E tests from add-tests command` 的测试文件提交
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/add-tests.md
</execution_context>

<context>
阶段：$ARGUMENTS

@.planning/STATE.md
@.planning/ROADMAP.md
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/add-tests.md 中的 add-tests 工作流。
保留所有工作流关卡（分类审批、测试计划审批、RED-GREEN 验证、差距报告）。
</process>
