---
name: gsd:audit-uat
description: 对所有未完成的 UAT 和验证项进行跨阶段审计
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---
<objective>
扫描所有阶段中待处理、已跳过、被阻塞以及需要人工处理的 UAT 项目。与代码库交叉引用以检测过时的文档。生成按优先级排序的人工测试计划。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/audit-uat.md
</execution_context>

<context>
核心规划文件在工作流中通过 CLI 加载。

**范围：**
Glob: .planning/phases/*/*-UAT.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>
