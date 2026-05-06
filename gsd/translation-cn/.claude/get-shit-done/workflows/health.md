<purpose>
验证 `.planning/` 目录的完整性并报告可操作的问题。检查缺失文件、无效配置、不一致状态和孤立计划。可选择修复可自动修复的问题。
</purpose>

<process>

<step name="parse_args">
解析参数中的 --repair、--backfill 和 --context 标志。如果设置了 --context，跳转到上下文检查步骤。
</step>

<step name="context_check">
仅在设置 --context 时运行。检查当前会话的 token 使用情况并计算上下文利用率。
</step>

<step name="run_health_check">
运行 `gsd-sdk query validate.health` 进行健康验证。解析 JSON 输出获取状态（healthy/degraded/broken）、错误、警告和信息。
</step>

<step name="format_output">
格式化并显示结果（状态横幅、错误、警告、信息）。如果有可修复的问题且未使用 --repair，建议运行修复。
</step>

<step name="offer_repair">
如果存在可修复的问题且未使用 --repair，询问用户是否要运行修复。
</step>

<step name="verify_repairs">
如果执行了修复，重新运行健康检查以确认问题已解决。
</step>

</process>

<error_codes>
包含错误代码表（E001-E005、W001-W009、W018-W019、I001 等）及描述和可修复性。
</error_codes>

<repair_actions>
列出可用的修复操作（createConfig、resetConfig、regenerateState、addNyquistKey、backfillMilestones）及其效果和风险。
</repair_actions>
