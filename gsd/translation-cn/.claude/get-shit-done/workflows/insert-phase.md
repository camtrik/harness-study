<purpose>
在现有整数阶段之间插入一个十进制阶段，用于里程碑中期发现的紧急工作。使用十进制编号（72.1、72.2 等）来保持计划阶段的逻辑顺序，同时适应紧急插入而不重新编号整个路线图。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="parse_arguments">
解析命令参数：第一个参数为要插入其后的整数阶段编号，其余参数为阶段描述。验证第一个参数是整数。
</step>

<step name="init_context">
加载阶段操作上下文，检查 roadmap 是否存在。
</step>

<step name="insert_phase">
将阶段插入委托给 `gsd-sdk query phase.insert`。CLI 处理：验证目标阶段存在、计算下一个十进制阶段编号、生成 slug、创建阶段目录、将阶段条目插入 ROADMAP.md 并附 (INSERTED) 标记。
</step>

<step name="update_project_state">
通过 SDK 处理程序更新 STATE.md（不使用原始 Edit/Write）以反映插入的阶段。
</step>

<step name="completion">
呈现完成摘要，包含十进制阶段编号、描述、目录和下一步建议。
</step>

</process>

<anti_patterns>
- 不要对里程碑末尾的计划工作使用此命令（使用 /gsd-add-phase）
- 不要在阶段 1 之前插入
- 不要重新编号现有阶段
- 不要修改目标阶段内容
</anti_patterns>

<success_criteria>
- [ ] `gsd-sdk query phase.insert` executed successfully
- [ ] Phase directory created
- [ ] Roadmap updated with (INSERTED) marker
- [ ] STATE.md updated
- [ ] User informed of next steps
</success_criteria>
