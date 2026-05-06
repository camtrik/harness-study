<purpose>
在适当深度级别执行发现。生成 DISCOVERY.md（对于级别 2-3），为 PLAN.md 创建提供信息。

由 plan-phase.md 的 mandatory_discovery 步骤调用，传入深度参数。

注意：对于全面的生态系统研究（"专家是如何构建这个的"），请改用 /gsd-plan-phase --research-phase，它会生成 RESEARCH.md。
</purpose>

<depth_levels>
**此工作流支持三个深度级别：**

| 级别 | 名称 | 时间 | 输出 | 适用场景 |
| ----- | ------------ | --------- | -------------------------------------------- | ----------------------------------------- |
| 1 | Quick Verify | 2-5 分钟 | 无文件，用已验证的知识继续 | 单个库，确认当前语法 |
| 2 | Standard | 15-30 分钟 | DISCOVERY.md | 在选项之间选择，新的集成 |
| 3 | Deep Dive | 1 小时以上 | 带有验证门的详细 DISCOVERY.md | 架构决策，新问题 |

**深度由 plan-phase.md 在路由到这里之前确定。**
</depth_levels>

<source_hierarchy>
**强制性：Context7 优先于 WebSearch**

Claude 的训练数据滞后 6-18 个月。始终验证。

1. **Context7 MCP 优先** — 最新文档，无幻觉
2. **官方文档** — 当 Context7 覆盖不足时
3. **WebSearch 最后** — 仅用于比较和趋势

参见 templates/discovery.md 获取完整协议。
</source_hierarchy>

<process>

<step name="determine_depth">
检查从 plan-phase.md 传入的深度参数并路由到适当级别。
</step>

<step name="level_1_quick_verify">
**级别 1：快速验证（2-5 分钟）**

适用于单个已知库，确认语法/版本仍然正确。通过 Context7 解析库、获取文档并验证。如果已验证则返回确认，如果发现疑虑则升级到级别 2。
</step>

<step name="level_2_standard">
**级别 2：标准发现（15-30 分钟）**

适用于在选项之间选择、新的外部集成。识别要发现的内容、通过 Context7 了解每个选项、查阅官方文档、通过 WebSearch 获取比较信息、交叉验证所有发现、创建 DISCOVERY.md（含推荐摘要和关键发现，置信度应为 MEDIUM-HIGH）。
</step>

<step name="level_3_deep_dive">
**级别 3：深入探究（1 小时以上）**

适用于架构决策、新问题、高风险选择。划定发现范围、全面进行 Context7 研究、深入阅读官方文档、通过 WebSearch 获取生态系统上下文、交叉验证所有发现、创建全面的 DISCOVERY.md（含质量报告和来源归属），如果置信度低则设置验证检查点。
</step>

<step name="confidence_gate">
检查置信度级别。如果 LOW，提供深入探究/继续/暂停选项。如果是 MEDIUM 则行内报告。如果是 HIGH 则直接继续。
</step>

</process>

<success_criteria>
覆盖三个深度级别各自的成功标准（快速验证、标准发现、深入探究）。
</success_criteria>
