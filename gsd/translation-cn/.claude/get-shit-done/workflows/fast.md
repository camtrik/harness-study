<purpose>
内联执行一个简单任务，无 subagent 开销。无 PLAN.md、无 Task 启动、无研究、无计划检查。只需：理解 → 执行 → 提交 → 记录。

适用于：修复拼写错误、更新配置值、添加缺失的导入、重命名变量、提交未提交的工作、添加 .gitignore 条目、更新版本号等任务。

对于需要多步骤规划或研究的任务，请使用 /gsd-quick。
</purpose>

<process>

<step name="parse_task">
解析 `$ARGUMENTS` 中的任务描述。如果为空，询问快速修复是什么。
</step>

<step name="scope_check">
验证任务确实是简单的（≤ 3 个文件编辑、≤ 1 分钟工作、无新依赖或架构更改、无需研究）。如果任务看起来不简单，重定向到 /gsd-quick。
</step>

<step name="execute_inline">
直接执行：读取相关文件 → 进行更改 → 验证更改有效。
</step>

<step name="commit">
使用常规提交格式原子提交更改。
</step>

<step name="log_to_state">
如果 STATE.md 存在，追加到"Quick Tasks Completed"表。
</step>

</process>

<guardrails>
- 永远不要启动 Task/subagent — 此工作流内联运行
- 永远不要创建 PLAN.md 或 SUMMARY.md 文件
- 永远不要运行研究或计划检查
- 如果任务超过 3 个文件编辑，停止并重定向到 /gsd-quick
- 如果不确定如何实现，停止并重定向到 /gsd-quick
</guardrails>

<success_criteria>
- [ ] 任务在当前上下文中完成（无 subagent）
- [ ] 使用常规消息进行原子 git 提交
- [ ] STATE.md 已更新（如果存在）
- [ ] 总操作在 2 分钟以内
</success_criteria>
