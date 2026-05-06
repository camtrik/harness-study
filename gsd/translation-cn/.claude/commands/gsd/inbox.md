---
name: gsd:inbox
description: 对照项目模板和贡献指南对开放的 GitHub issues 和 PRs 进行分类审查
argument-hint: "[--issues] [--prs] [--label] [--close-incomplete] [--repo owner/repo]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Grep
  - Glob
  - AskUserQuestion
---
<objective>
一键分类处理项目的 GitHub 收件箱。获取所有开放的 issues 和 PRs，
逐一对照相应的模板要求进行审查（feature、enhancement、
bug、chore、fix PR、enhancement PR、feature PR），报告完整性和合规性，
并可选择性地应用标签或关闭不符合要求的提交。

**流程：** 检测仓库 → 获取开放的 issues + PRs → 按类型分类 → 对照模板审查 → 报告发现 → 可选操作（打标签、评论、关闭）
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/inbox.md
</execution_context>

<context>
**标志：**
- `--issues` — 仅审查 issues（跳过 PRs）
- `--prs` — 仅审查 PRs（跳过 issues）
- `--label` — 审查后自动应用推荐标签
- `--close-incomplete` — 关闭不符合模板要求的 issues/PRs（附带解释性评论）
- `--repo owner/repo` — 覆盖自动检测的仓库（默认为当前 git remote）
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/inbox.md 中的 inbox 工作流。
从参数解析标志并传递给工作流。
</process>
