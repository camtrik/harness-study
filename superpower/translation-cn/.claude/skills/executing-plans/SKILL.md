---
name: executing-plans
description: 当你有一份书面的实施计划需要在独立会话中执行并设置审查检查点时使用
---

# 执行计划

## 概述

加载计划，进行批判性审查，执行所有任务，完成后报告。

**开始时声明：** "我正在使用 executing-plans skill 来实施此计划。"

**注意：** 请告知你的人类搭档，Superpowers 在拥有 subagent 访问权限时效果更好。如果在支持 subagent 的平台（如 Claude Code 或 Codex）上运行，其工作质量会显著提高。如果 subagent 可用，请使用 superpowers:subagent-driven-development 而非此 skill。

## 执行流程

### 第 1 步：加载并审查计划
1. 阅读计划文件
2. 批判性审查——识别关于计划的任何疑问或顾虑
3. 如果有顾虑：在开始之前向你的人类搭档提出
4. 如果没有顾虑：创建 TodoWrite 并继续

### 第 2 步：执行任务

对于每个任务：
1. 标记为 in_progress
2. 严格按每个步骤执行（计划包含小步步骤）
3. 按指定要求运行验证
4. 标记为已完成

### 第 3 步：完成开发

所有任务完成并验证通过后：
- 声明："我正在使用 finishing-a-development-branch skill 来完成此项工作。"
- **必须使用的 SUB-SKILL：** 使用 superpowers:finishing-a-development-branch
- 遵循该 skill 进行测试验证、提供选项、执行选择

## 何时停止并请求帮助

**立即停止执行的时机：**
- 遇到阻塞（缺少依赖、测试失败、指令不清晰）
- 计划存在阻止启动的关键缺陷
- 你不理解某条指令
- 验证反复失败

**请求澄清，而不是猜测。**

## 何时回溯之前的步骤

**回到审查阶段（第 1 步）的时机：**
- 搭档根据你的反馈更新了计划
- 基本方法需要重新思考

**不要强行通过阻塞**——停下来询问。

## 注意事项

- 首先要批判性审查计划
- 严格遵循计划步骤
- 不要跳过验证
- 计划要求引用 skill 时务必引用
- 遇到阻塞就停下，不要猜测
- 未经用户明确同意，绝不在 main/master 分支上开始实施

## 集成

**必须使用的工作流 skill：**
- **superpowers:using-git-worktrees** - 确保隔离的工作空间（创建新的或验证现有的）
- **superpowers:writing-plans** - 创建此 skill 要执行的计划
- **superpowers:finishing-a-development-branch** - 所有任务完成后完成开发
