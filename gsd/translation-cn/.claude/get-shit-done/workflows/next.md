<purpose>
检测当前项目状态并自动推进到下一个逻辑 GSD 工作流步骤（讨论 → 规划 → 执行 → 验证 → 完成）。
</purpose>
<process>
<step name="detect_state">读取项目状态确定当前位置。</step>
<step name="safety_gates">运行硬停止检查（未解决的检查点、错误状态、未检查的验证失败、先前阶段完整性扫描）。</step>
<step name="spike_sketch_notice">检查待处理的 spike/sketch 工作并给出通知。</step>
<step name="determine_next_action">根据状态应用路由规则（无阶段→讨论、无上下文→讨论、有上下文无计划→规划、有计划无摘要→执行、全部有摘要→验证、阶段完成→推进、全部完成→完成里程碑、暂停→恢复）。</step>
<step name="show_and_execute">显示决定并立即调用确定的命令。</step>
</process>
