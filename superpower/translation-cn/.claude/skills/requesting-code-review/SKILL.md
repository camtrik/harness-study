---
name: requesting-code-review
description: 在完成任务、实现主要功能或合并前使用，用于验证工作是否满足需求
---

# 请求代码审查

派发代码审查 subagent，以便在问题扩散之前捕获它们。审查者会收到精确构建的评估上下文——而不是你会话的历史记录。这使得审查者专注于工作产出而非你的思考过程，同时为你自己的上下文保留空间以继续工作。

**核心原则：** 早审查，常审查。

## 何时请求审查

**强制性：**
- subagent 驱动开发中每个任务完成后
- 完成主要功能后
- 合并到 main 之前

**可选但推荐：**
- 卡住时（获得新视角）
- 重构之前（基线检查）
- 修复复杂 bug 之后

## 如何请求审查

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码审查 subagent：**

使用 Task tool，类型为 `general-purpose`，填写 `code-reviewer.md` 中的模板

**占位符：**
- `{DESCRIPTION}` - 已构建内容的简要摘要
- `{PLAN_OR_REQUIREMENTS}` - 应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交

**3. 根据反馈采取行动：**
- 立即修复严重问题
- 在继续之前修复重要问题
- 记录次要问题待后续处理
- 如果审查者判断有误，予以反驳（附上理由）

## 示例

```
[刚刚完成任务 2：添加验证功能]

你：在继续之前让我请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发代码审查 subagent]
  DESCRIPTION：添加了 verifyIndex() 和 repairIndex()，包含 4 种问题类型
  PLAN_OR_REQUIREMENTS：docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA：a7981ec
  HEAD_SHA：3df7661

[Subagent 返回]：
  优点：架构干净，测试真实
  问题：
    重要：缺少进度指示器
    次要：报告间隔使用魔术数字 (100)
  评估：可以继续

你：[修复进度指示器]
[继续任务 3]
```

## 与工作流的集成

**Subagent 驱动开发：**
- 每个任务后审查
- 在问题累积之前捕获
- 在移动到下一个任务之前修复

**执行计划：**
- 每个任务后或自然检查点处审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 红线

**绝不：**
- 因为"很简单"而跳过审查
- 忽略严重问题
- 带着未修复的重要问题继续
- 与合理的技术反馈争论

**如果审查者判断有误：**
- 用技术理由反驳
- 展示能证明其正常工作的代码/测试
- 请求澄清

模板参见：requesting-code-review/code-reviewer.md
