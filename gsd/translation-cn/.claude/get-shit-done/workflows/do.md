<purpose>
分析用户的自由格式文本并将其路由到最合适的 GSD 命令。这是一个分派器 — 它从不自己执行工作。将用户意图匹配到最佳命令，确认路由，然后交接。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="validate">
**检查输入。** 如果 `$ARGUMENTS` 为空，通过 AskUserQuestion 询问用户描述任务。
</step>

<step name="check_project">
**检查项目是否存在。** 跟踪 `.planning/` 是否存在 — 某些路由需要它，其他不需要。
</step>

<step name="route">
**将意图匹配到命令。** 根据路由规则评估 `$ARGUMENTS`，应用第一个匹配的规则。路由表映射任务描述到 GSD 命令，并附原因。需要 `.planning/` 目录的路由会在项目不存在时建议先运行 `/gsd-new-project`。如果有歧义，询问用户选择前 2-3 个选项。
</step>

<step name="display">
**显示路由决策。** 以格式化横幅显示输入、路由到的命令和原因。
</step>

<step name="dispatch">
**调用所选命令。** 运行选定的 `/gsd-*` 命令，将 `$ARGUMENTS` 作为参数传入。如果命令需要阶段编号但未提供，则从上下文提取或询问用户。调用后停止。
</step>

</process>

<success_criteria>
- [ ] Input validated (not empty)
- [ ] Intent matched to exactly one GSD command
- [ ] Ambiguity resolved via user question (if needed)
- [ ] Project existence checked for routes that require it
- [ ] Routing decision displayed before dispatch
- [ ] Command invoked with appropriate arguments
- [ ] No work done directly — dispatcher only
</success_criteria>
