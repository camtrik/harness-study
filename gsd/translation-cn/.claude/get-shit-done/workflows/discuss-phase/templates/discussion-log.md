# DISCUSSION-LOG.md 模板 — 用于讨论阶段 git_commit 步骤

> **延迟加载。** 仅在 `workflows/discuss-phase.md` 的 `git_commit` 步骤内读取此文件，紧接在写入 `${phase_dir}/${padded_phase}-DISCUSSION-LOG.md` 之前。

## 目的

供人工审查的审计追踪（合规、学习、回顾）。不被下游 agent 消费 — 那些只读取 CONTEXT.md。

## 模板正文

```markdown
# Phase [X]: [Name] - Discussion Log

> **仅供审计追踪。** 不要用作规划、研究或执行 agent 的输入。
> 决策已捕获在 CONTEXT.md 中 — 此日志保留了考虑的替代方案。

**Date:** [ISO 日期]
**Phase:** [阶段编号]-[阶段名称]
**Areas discussed:** [逗号分隔列表]

---

[对于每个讨论过的灰色区域：]

## [区域名称]

| Option | Description | Selected |
|--------|-------------|----------|
| [选项 1] | [来自 AskUserQuestion 的描述] | |
| [选项 2] | [描述] | ✓ |
| [选项 3] | [描述] | |

**用户的选择：** [选定的选项或自由文本回复]
**备注：** [用户提供的任何澄清、后续上下文或理由]

---

[每个区域重复]

## Claude's Discretion

[列出用户说"you decide"或交给 Claude 决定的领域]

## Deferred Ideas

[讨论中提到并被记录为未来阶段想法的内容]
```
