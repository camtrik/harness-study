<purpose>
列出所有待处理的待办事项，允许选择，加载所选待办的完整上下文，并路由到适当的操作。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="init_context">
加载待办上下文：

```bash
INIT=$(gsd-sdk query init.todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

提取 `todo_count`、`todos`、`pending_dir`。如果没有待处理的待办，显示相应消息并退出。
</step>

<step name="parse_filter">
检查参数中的区域过滤器：`/gsd-capture --list` 显示全部，`/gsd-capture --list api` 仅过滤 api 区域。
</step>

<step name="list_todos">
使用 init 上下文中的 todos 数组，解析并显示为编号列表。
</step>

<step name="handle_selection">
等待用户回复编号。有效则加载选中待办；无效则提示重新输入。
</step>

<step name="load_context">
完全读取待办文件，显示标题、区域、创建时间、文件和问题/解决方案部分。
</step>

<step name="check_roadmap">
检查待办是否与路线图中的阶段匹配（区域匹配、文件重叠等）。
</step>

<step name="offer_actions">
根据是否映射到路线图阶段，提供不同的操作选项（开始工作、添加到阶段计划、创建阶段、头脑风暴、放回列表）。
</step>

<step name="execute_action">
执行所选操作（移动待办到已完成、提交、状态更新等）。
</step>

<step name="update_state">
重新运行 init todos 获取更新后的计数，更新 STATE.md。
</step>

<step name="git_commit">
如果待办已移至完成，提交更改。

```bash
gsd-sdk query commit "docs: start work on todo - [title]" --files .planning/todos/completed/[filename] .planning/STATE.md
```
</step>

</process>

<success_criteria>
- [ ] 所有待处理待办按标题、区域、时间列出
- [ ] 区域过滤器已应用（如果指定）
- [ ] 选中待办的完整上下文已加载
- [ ] 路线图上下文已检查阶段匹配
- [ ] 适当的操作已提供
- [ ] 选中操作已执行
- [ ] 待办计数变更时 STATE.md 已更新
- [ ] 更改已提交到 git（如果待办移至完成）
</success_criteria>
