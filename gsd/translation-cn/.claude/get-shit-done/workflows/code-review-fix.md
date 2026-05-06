<purpose>
自动化修复 REVIEW.md 中的问题。验证阶段、检查配置门、验证 REVIEW.md 存在且有可修复的问题、启动 gsd-code-fixer agent、处理 --auto 迭代循环（上限 3 次）、最终一次性提交 REVIEW-FIX.md，并呈现结果。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<available_agent_types>
- gsd-code-fixer：对代码审查发现项应用修复
- gsd-code-reviewer：审查源文件中的 bug 和问题
</available_agent_types>

<process>

<step name="initialize">
解析参数并加载项目状态，包括阶段参数、init JSON 信息。进行输入清理（验证阶段编号格式）。在配置门检查之前验证阶段是否存在。解析 `--all` 和 `--auto` 可选标志。
</step>

<step name="check_config_gate">
检查 `workflow.code_review` 配置是否启用。如果禁用则跳过。重用 `workflow.code_review` 配置键而非引入新的 `workflow.code_review_fix` 键。
</step>

<step name="check_review_exists">
验证 REVIEW.md 存在。如果不存在则报错。不自动运行代码审查 — 需要明确的用户操作。
</step>

<step name="check_review_status">
解析 REVIEW.md frontmatter 以检查状态。如果状态为 clean 或 skipped，则退出。提取审查深度和原始审查文件列表用于 --auto 重新审查。
</step>

<step name="spawn_fixer">
启动 gsd-code-fixer agent 并传入审查路径、阶段目录和修复范围配置。agent 读取 REVIEW.md 发现项、应用修复、原子提交每个修复并写入 REVIEW-FIX.md。

> **编排器规则 — CODEX 运行时**：调用 Task() 后，立即停止此任务的工作。等待 subagent 返回结果。

处理 agent 失败情况（部分成功感知）。
</step>

<step name="auto_iteration_loop">
仅在 AUTO_MODE 时运行。迭代语义：初始修复遍为迭代 1，循环运行迭代 2 到 MAX_ITERATIONS（上限 3）。每次迭代：重新审查 → 检查状态 → 仍有问题则重新修复。关键设计决策：重新审查范围使用持久化的文件范围，REVIEW.md 被覆盖（最新状态），REVIEW-FIX.md 最后一次性提交。
</step>

<step name="commit_fix_report">
在所有迭代完成后，验证并提交 REVIEW-FIX.md（仅一次）。验证 frontmatter 有效（包含 status 字段），如果 `commit_docs` 为 true 则提交。
</step>

<step name="present_results">
解析 REVIEW-FIX.md frontmatter 并向用户呈现格式化的摘要。根据状态（all_fixed/partial/none_fixed）提供不同的下一步建议。
</step>

</process>

<platform_notes>
**Windows：** 此工作流使用 bash 特性。在 Windows 上需要 Git Bash 或 WSL，不支持原生 PowerShell。
</platform_notes>

<success_criteria>
- [ ] 阶段在配置门检查之前验证
- [ ] 配置门已检查（workflow.code_review）
- [ ] REVIEW.md 存在性已验证
- [ ] REVIEW.md 状态已检查
- [ ] Agent 已使用正确配置启动
- [ ] --auto 迭代循环遵守 3 次迭代上限
- [ ] --auto 重新审查使用持久化文件范围
- [ ] REVIEW-FIX.md 在多次迭代后仅提交一次
- [ ] 结果以内联方式呈现并附下一步建议
</success_criteria>
