<purpose>
从项目路线图中移除一个未开始的未来阶段，删除其目录，重新编号所有后续阶段以维护清晰的线性序列，并提交更改。
</purpose>
<process>
<step name="parse_arguments">解析要移除的阶段编号。</step>
<step name="init_context">加载阶段操作上下文。</step>
<step name="validate_future_phase">验证阶段是未来的（未开始）。</step>
<step name="confirm_removal">呈现移除摘要并确认。</step>
<step name="execute_removal">委托给 `gsd-sdk query phase.remove`。</step>
<step name="commit">提交移除。</step>
</process>
