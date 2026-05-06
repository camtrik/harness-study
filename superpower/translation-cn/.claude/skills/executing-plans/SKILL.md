---
name: executing-plans
description: 当你有书面实施计划在单独的会话中执行并带有审查检查点时使用
---

# 执行计划

## 概述

加载计划、批判性审查、执行所有任务、完成时报告。

**开始时宣布：** "我正在使用 executing-plans skill 来实施此计划。"

**注意：** 告诉你的人类搭档，Superpower 在访问 subagent 的情况下效果更好。如果在支持 subagent 的平台（如 Claude Code 或 Codex）上运行，其工作质量将显著更高。如果 subagent 可用，请使用 superpowers:subagent-driven-development 而不是此 skill。

## 流程

### 步骤 1：加载和审查计划
1. 阅读计划文件
2. 批判性审查——识别对计划的任何问题或疑虑
3. 如果有疑虑：在开始之前与你的人类搭档提出
4. 如果没有疑虑：创建 TodoWrite 并继续

### 步骤 2：执行任务

对于每个任务：
1. 标记为 in_progress
2. 完全按照每个步骤（计划有小的步骤）
3. 按照规定运行验证
4. 标记为已完成

### 步骤 3：完成开发

所有任务完成并验证后：
- 宣布： "我正在使用 finishing-a-development-branch skill 来完成这项工作。"
- **必需的子 skill：** 使用 superpowers:finishing-a-development-branch
- 遵循该 skill 验证测试、展示选项、执行选择

## 何时停止并寻求帮助

**立即停止执行时机：**
- 遇到阻塞器（缺少依赖项、测试失败、指令不清楚）
- 计划有关键差距阻止开始
- 你不理解指令
- 验证反复失败

**要求澄清而不是猜测。**

## 何时重新审视早期步骤

**返回审查（步骤 1）时机：**
- 合作伙伴根据你的反馈更新计划
- 根本方法需要重新思考

**不要强行通过阻塞器**——停止并询问。

## 记住
- 首先批判性审查计划
- 完全按照计划步骤
- 不要跳过验证
- 当计划说时参考 skill
- 被阻止时停止，不要猜测
- 在没有明确的用户同意之前，永远不要在 main/master 分支上开始实施

## 集成

**必需的工作流 skill：**
- **superpowers:using-git-worktrees**——确保隔离的工作区（创建一个或验证现有）
- **superpowers:writing-plans**——创建此 skill 执行的计划
- **superpowers:finishing-a-development-branch**——在所有任务后完成开发
