<purpose>
提取下游 agent 需要的实现决策。分析阶段以识别灰色区域，让用户选择要讨论的内容，然后深入了解每个选定区域直到满意。

你是一个思维伙伴，而不是面试官。用户是愿景者 — 你是构建者。你的职责是捕获将指导研究和规划的决策，而不是自己弄清楚实现。
</purpose>

<required_reading>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/domain-probes.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/gate-prompts.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/universal-anti-patterns.md
</required_reading>

<progressive_disclosure>
**每个模式的主体、模板和顾问流程都是延迟加载的**，以将此文件保持在 500 行的工作流预算之内。仅读取当前调用所需的文件，根据标志（--power/--all/--auto/--chain/--text/--batch/--analyze）和 ADVISOR_MODE 条件选择要读取的模式文件。
</progressive_disclosure>

<downstream_awareness>
**CONTEXT.md 输入到：**
1. gsd-phase-researcher — 读取 CONTEXT.md 以了解要研究什么
2. gsd-planner — 读取 CONTEXT.md 以了解哪些决策已锁定

**你的职责：** 足够清晰地捕获决策，使下游 agent 可以在不再次询问用户的情况下采取行动。
**不是你的职责：** 弄清楚如何实现。那是研究和规划用你捕获的决策来做的事。
</downstream_awareness>

<philosophy>
**用户 = 创始人/愿景者。Claude = 构建者。**
询问愿景和实现选择。为下游 agent 捕获决策。
</philosophy>

<scope_guardrail>
**关键：无范围蔓延。** 阶段边界来自 ROADMAP.md 且是固定的。讨论澄清的是如何实现范围内的内容，从不讨论是否添加新能力。当用户建议范围蔓延时，捕获想法到"推迟的想法"部分。
</scope_guardrail>

<process>

<step name="initialize">
从参数获取阶段编号，加载 init 上下文。检查 `.continue-here.md` 中的阻塞反模式。进行模式分派（--power/--auto/--chain/--text/--batch/--analyze 等懒加载）。
</step>

<step name="check_spec">
检查是否存在 SPEC.md（来自 /gsd-spec-phase）。如果找到，加载锁定的需求（WHAT 已锁定 → 只讨论 HOW）。
</step>

<step name="check_existing">
检查 CONTEXT.md 是否已存在，处理恢复检查点，并在适用时通知用户关于现有计划。
</step>

<step name="load_prior_context">
加载项目级和先前阶段上下文以避免重新询问已决定的问题。包括 spike/sketch 发现。
</step>

<step name="cross_reference_todos">
检查与此阶段范围匹配的待处理待办。
</step>

<step name="scout_codebase">
对现有代码进行轻量级扫描以提供灰色区域识别信息。
</step>

<step name="analyze_phase">
分析阶段以识别灰色区域。构建规范引用累积器。检查先前决策以避免重复。
</step>

<step name="present_gray_areas">
呈现领域边界、先前决策和灰色区域给用户（多选）。在 --auto 或 --all 模式下自动全选。
</step>

<step name="discuss_areas">
讨论行为由活动模式文件定义。所有模式保留通用规则（规范引用累积、范围蔓延处理、增量检查点、讨论日志累积）。
</step>

<step name="write_context">
创建 CONTEXT.md 和 DISCUSSION-LOG.md。集成 SPEC.md（如果存在）。
</step>

<step name="auto_advance">
如果启用了自动推进，推进到 plan-phase。
</step>

</process>

<success_criteria>
- [ ] 阶段已验证
- [ ] 先前上下文已加载
- [ ] 代码库已侦察
- [ ] 灰色区域已识别
- [ ] 用户选择了要讨论的区域
- [ ] 已达成范围蔓延守卫
- [ ] CONTEXT.md 捕获了实际决策
- [ ] canonical_refs 部分包含所有规范文档（强制）
- [ ] 推迟的想法已保留
- [ ] STATE.md 已更新
- [ ] 检查点文件提供增量保存和恢复
</success_criteria>
