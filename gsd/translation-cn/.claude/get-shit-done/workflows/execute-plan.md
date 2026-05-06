<purpose>
执行阶段计划（PLAN.md）并创建结果摘要（SUMMARY.md）。
</purpose>

<required_reading>
读取 STATE.md 获取项目上下文。读取 config.json 获取规划行为设置。

@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/git-integration.md
</required_reading>

<available_agent_types>
- gsd-executor — 执行计划任务、提交、创建 SUMMARY.md
</available_agent_types>

<process>

<step name="init_context">
加载执行上下文（仅路径以最小化编排器上下文）。提取 executor_model、commit_docs、sub_repos、phase_dir、plans、summaries 等。
</step>

<step name="identify_plan">
找到第一个没有匹配 SUMMARY 的 PLAN。支持十进制阶段。
</step>

<step name="record_start_time">
记录计划开始时间。
</step>

<step name="parse_segments">
计算任务数量，确定内联阈值，根据检查点类型路由执行模式（Pattern A 自主、Pattern B 分段、Pattern C 内联）。
</step>

<step name="init_agent_tracking">
初始化 agent 追踪（agent-history.json、current-agent-id.txt）。如果在中断后有未完成的 agent，允许恢复。
</step>

<step name="segment_execution">
Pattern B 仅 — 对 verify-only 检查点执行逐段执行。在分段间启动 subagent 并聚合结果。
</step>

<step name="load_prompt">
读取 PLAN.md 作为执行指令。如果计划包含 `<interfaces>` 块，直接使用预提取的类型定义。
</step>

<step name="execute">
按任务执行，处理 TDD（RED-GREEN-REFACTOR）、检查点、验收标准验证门和认证门。遵循偏差规则。
</step>

<authentication_gates>
认证错误不是失败 — 它们是预期的交互点。识别后创建 human-action 检查点。
</authentication_gates>

<deviation_rules>
偏差规则：bugs/缺失/阻塞 → 自动修复（限 3 次）。架构更改 → 停止并等待用户批准。
</deviation_rules>

<step name="create_summary">
创建 SUMMARY.md，使用模板填写所有 frontmatter 字段（phase、plan、key-files、key-decisions、requirements-completed、持续时间等）。
</step>

<step name="update_current_position">
更新 STATE.md（并行模式下跳过 — 编排器处理）。
</step>

<step name="git_commit_metadata">
提交计划元数据。并行模式下排除 STATE.md 和 ROADMAP.md（编排器提交这些）。
</step>

<step name="offer_next">
根据剩余计划、阶段完成情况或里程碑完成情况提供下一步建议。
</step>

</process>

<success_criteria>
- 所有 PLAN.md 中的任务已完成
- 所有验证通过
- SUMMARY.md 已创建并包含实质性内容
- STATE.md 已更新（除非并行模式）
- ROADMAP.md 已更新（除非并行模式）
- 提交已完成
</success_criteria>
