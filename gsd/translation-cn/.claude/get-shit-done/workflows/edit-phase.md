<purpose>
编辑 ROADMAP.md 中现有阶段的任何字段。阶段编号和位置始终保留。除非传递 --force，否则对进行中和已完成的阶段进行保护。在写入之前验证 depends_on 引用。显示差异并在写入前请求确认。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="parse_arguments">
解析命令参数：第一个参数为要编辑的阶段编号，可选标志 --force。
</step>

<step name="init_context">
加载阶段操作上下文，检查 roadmap 是否存在。
</step>

<step name="load_phase">
从 ROADMAP.md 读取当前阶段部分，提取阶段名称、目标、成功标准、section 文本等字段（包括 depends_on 和 requirements）。
</step>

<step name="check_phase_status">
确定阶段状态（completed/in_progress/future）。如果状态为 in_progress 或 completed 且没有 --force，则阻止编辑。
</step>

<step name="present_current_values">
显示当前阶段字段。询问用户要做什么：编辑特定字段、从澄清的意图重新生成所有字段、或取消。
</step>

<step name="collect_edits">
根据用户选择收集编辑内容（单字段编辑或基于意图的完整重新生成）。
</step>

<step name="validate_depends_on">
如果更新了 depends_on，验证所有引用的阶段编号是否存在于 ROADMAP.md 中（不能自引用）。
</step>

<step name="show_diff_and_confirm">
构建更新的阶段部分，显示差异，等待用户确认。
</step>

<step name="write_updated_phase">
将更新的阶段原位置写回 ROADMAP.md。更新 STATE.md 的路线图演进记录。
</step>

<step name="completion">
呈现完成摘要，显示更改的字段和下一步建议。
</step>

</process>

<anti_patterns>
- 不要重新编号阶段 — 编号和位置必须原样保留
- 编辑一个阶段时不要修改其他阶段
- 不要跳过 depends_on 验证（无效引用阻止写入）
- 未显示差异并获得确认前不要写入
- 没有 --force 不要编辑 in_progress/completed 阶段
</anti_patterns>

<success_criteria>
- [ ] 阶段已从 ROADMAP.md 找到并加载
- [ ] 状态检查已执行
- [ ] 当前值已向用户呈现
- [ ] 用户选择了编辑模式
- [ ] depends_on 引用已验证
- [ ] 差异已显示并获得用户确认
- [ ] 更新的阶段已原位写回
- [ ] STATE.md 路线图演进已更新
- [ ] 用户已被告知下一步
</success_criteria>
