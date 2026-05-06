<purpose>
诊断 UAT 差距并寻找根本原因的并行 debug agent 编排器。

在 UAT 发现差距后，每个差距启动一个 debug agent。每个 agent 使用从 UAT 预填的症状自主调查。收集根本原因，用诊断结果更新 UAT.md 差距，然后将实际的诊断结果交给 plan-phase --gaps 处理。

编排器保持精简：解析差距，启动 agent，收集结果，更新 UAT。
</purpose>

<available_agent_types>
- gsd-debugger — 诊断和修复问题
</available_agent_types>

<paths>
DEBUG_DIR=.planning/debug
</paths>

<core_principle>
**在规划修复之前先诊断。**

UAT 告诉我们什么坏了（症状）。Debug agent 找出为什么坏（根本原因）。plan-phase --gaps 然后基于实际原因创建针对性的修复，而不是猜测。
</core_principle>

<process>

<step name="parse_gaps">
从 UAT.md 的 Gaps 部分（YAML 格式）提取差距。构建差距列表，包含 truth、severity、test_num、reason 等信息。
</step>

<step name="report_plan">
读取 worktree 配置，向用户报告诊断计划（显示差距表和严重级别）。
</step>

<step name="spawn_agents">
加载 agent skills，并行启动 debug agent（每个差距一个）。填充 debug-subagent-prompt 模板，传入 truth、expected、actual、errors、slug 等变量。

> **编排器规则 — CODEX 运行时**：调用 Task() 后，等待所有 subagent 返回后再继续。
</step>

<step name="collect_results">
从 agent 收集根本原因结果。解析 ROOT CAUSE FOUND 或 INVESTIGATION INCONCLUSIVE 返回值。提取 root_cause、files、debug_path、suggested_fix。
</step>

<step name="update_uat">
用诊断结果更新 UAT.md 差距（添加 root_cause、artifacts、missing、debug_session），将状态更新为 diagnosed，并提交更新的 UAT.md。
</step>

<step name="report_results">
报告诊断结果表并交给 verify-work 编排器进行自动规划。
</step>

</process>

<success_criteria>
- [ ] 从 UAT.md 解析出差距
- [ ] Debug agent 并行启动
- [ ] 从所有 agent 收集到根本原因
- [ ] UAT.md 差距已更新 artifact 和 missing 字段
- [ ] 调试会话已保存到 ${DEBUG_DIR}/
- [ ] 交给 verify-work 进行自动规划
</success_criteria>
