<purpose>
在规划前呈现 Claude 对阶段的假设，使用户能够及早纠正误解。

与 discuss-phase 的关键区别：这是对 Claude 认为的**分析**，而不是对用户知道的**收集**。不生成文件输出 — 纯粹是对话式的以促进讨论。
</purpose>

<process>
<step name="validate_phase">验证阶段存在于路线图中。</step>
<step name="analyze_phase">基于路线图描述和项目上下文，在五个领域中识别假设：技术方案、实现顺序、范围边界、风险领域、依赖关系。用置信度标记假设。</step>
<step name="present_assumptions">以清晰的格式呈现假设并询问用户意见。</step>
<step name="gather_feedback">确认用户的纠正或确认。</step>
<step name="offer_next">呈现下一步选项（讨论上下文/规划阶段/重新审视假设/完成）。</step>
</process>
