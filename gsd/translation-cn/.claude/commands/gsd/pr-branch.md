---
name: gsd:pr-branch
description: 通过过滤掉 .planning/ 提交来创建干净的 PR 分支 — 为代码审查做好准备
argument-hint: "[target branch, default: main]"
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

<objective>
通过从当前分支中过滤掉 .planning/ 提交，创建一个适合 Pull Request 的干净分支。
审查者只看到代码变更，而非 GSD 规划制品。

这解决了 PR 差异文件中混杂着与代码审查无关的 PLAN.md、SUMMARY.md、STATE.md
变更的问题。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/pr-branch.md
</execution_context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/pr-branch.md 中的 pr-branch 工作流。
</process>
