---
name: requesting-code-review
description: 在完成任务、实施主要功能或合并之前使用，以验证工作满足需求
---

# 请求代码审查

派遣代码审查者 subagent 以在问题级联之前捕获它们。审查者获得精确制作的评估上下文——绝不是你会话的历史记录。这使审查者专注于工作产品，而不是你的思考过程，并保留你自己的上下文以继续工作。

**核心原则：** 早审查，常审查。

## 何时请求审查

**必须：**
- 在 subagent-driven development 中的每个任务之后
- 完成主要功能后
- 合并到 main 之前

**可选但有价值：**
- 卡住时（新视角）
- 重构之前（基线检查）
- 修复复杂 bug 之后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣代码审查者 subagent：**

使用 Task 工具的 `general-purpose` 类型，填写 `code-reviewer.md` 中的模板

**占位符：**
- `{DESCRIPTION}` - 你构建内容的简要摘要
- `{PLAN_OR_REQUIREMENTS}` - 应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交

**3. 对反馈采取行动：**
- 立即修复关键问题
- 在继续之前修复重要问题
- 记录次要问题以备后用
- 如果审查者错误则反驳（附带理由）

## 示例

```
[刚刚完成任务 2：添加验证函数]

你：让我在继续之前请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派遣代码审查者 subagent]
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，包含 4 个问题类型
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[Subagent 返回]：
  优势：干净的架构，真实的测试
  问题：
    重要：缺少进度指示器
    次要：报告间隔的魔术数字（100）
  评估：可以继续

你：[修复进度指示器]
[继续任务 3]
```

## 与工作流集成

**Subagent-Driven Development：**
- 每个任务后审查
- 在问题复合之前捕获它们
- 在移动到下一个任务之前修复

**执行计划：**
- 在每个任务后或自然检查点审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 警示信号

**绝不：**
- 因为"很简单"而跳过审查
- 忽略关键问题
- 在未修复重要问题的情况下继续
- 与有效的技术反馈争论

**如果审查者错误：**
- 用技术理由反驳
- 显示证明它工作的代码/测试
- 请求澄清

参见模板：requesting-code-review/code-reviewer.md
