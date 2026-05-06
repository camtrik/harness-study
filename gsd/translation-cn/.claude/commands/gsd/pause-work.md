---
name: gsd:pause-work
description: 在阶段中途暂停工作时创建上下文交接文件
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
创建 `.continue-here.md` 交接文件，以保存跨会话的完整工作状态。

路由到 pause-work 工作流，该工作流处理：
- 从最近文件中检测当前阶段
- 完整状态收集（位置、已完成工作、剩余工作、决策、阻塞项）
- 创建包含所有上下文部分的交接文件
- Git WIP 提交
- 恢复说明
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/pause-work.md
</execution_context>

<context>
状态和阶段进度在工作流中通过针对性读取收集。
</context>

<process>
**遵循 pause-work 工作流**，来自 `@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/pause-work.md`。

工作流处理所有逻辑，包括：
1. 阶段目录检测
2. 带用户澄清的状态收集
3. 带时间戳的交接文件写入
4. Git 提交
5. 带恢复说明的确认
</process>
