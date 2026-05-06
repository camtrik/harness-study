<purpose>
审查阶段中变更的源文件，检查是否存在 bug、安全问题和代码质量问题。计算文件范围（--files 覆盖 > SUMMARY.md > git diff 回退），检查配置门，启动 gsd-code-reviewer agent，提交 REVIEW.md，并向用户呈现结果。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<available_agent_types>
- gsd-code-reviewer：审查源文件中的 bug 和质量问题
</available_agent_types>

<process>

<step name="initialize">
解析参数并加载项目状态，包括阶段验证（在配置门检查之前）、深度标志解析（--depth=quick|standard|deep）和文件覆盖标志解析（--files=...）。
</step>

<step name="check_config_gate">
检查 `workflow.code_review` 配置。如果明确设为 false 则跳过，否则默认为 true（在阶段验证之后检查，以便先显示无效阶段错误）。
</step>

<step name="resolve_depth">
按优先级确定审查深度：1. DEPTH_OVERRIDE（--depth 标志），2. 配置值，3. 默认 "standard"。验证深度值为 quick/standard/deep 之一。
</step>

<step name="compute_file_scope">
三级范围确定，具有明确优先级：

**第一级 — --files 覆盖（最高优先级）：** 如果提供了 --files，使用指定的文件（带路径遍历保护）。

**第二级 — SUMMARY.md 提取：** 如果未提供 --files，从阶段 SUMMARY.md 文件的 key_files.created 和 key_files.modified 中提取文件路径。

**第三级 — Git diff 回退：** 如果没有 SUMMARY.md 文件或未从中提取到文件，使用 git diff（排除规划产物和锁文件）。安全关闭 — 如果没有阶段提交，则不使用任意 HEAD~N。

对结果应用后处理：排除规划产物、过滤已删除文件、去重、排序。如果文件数量过多则发出警告。
</step>

<step name="check_empty_scope">
如果 REVIEW_FILES 为空，跳过审查（不启动 agent，不创建 REVIEW.md）。
</step>

<step name="spawn_reviewer">
计算审查输出路径并启动 gsd-code-reviewer agent，传入文件列表、深度和输出路径配置。agent 将发现项写入 REVIEW.md。

> **编排器规则 — CODEX 运行时**：调用 Task() 后，等待 subagent 返回结果。

处理 agent 失败情况：不继续到提交步骤，不创建部分或空的 REVIEW.md。
</step>

<step name="commit_review">
Agent 成功完成后，验证 REVIEW.md 已创建且具有有效 frontmatter（包含 status 字段）。如果有效且 `commit_docs` 为 true，则提交。
</step>

<step name="present_results">
读取 REVIEW.md YAML frontmatter 提取发现项计数（critical、warning、info、total）。以内联方式向用户呈现格式化摘要。如果发现项 > 0，列出前 3 个问题并提供下一步建议。
</step>

</process>

<platform_notes>
**Windows：** 需要 Git Bash 或 WSL。**macOS：** 使用 bash 3.2 兼容语法（不使用 mapfile），路径验证使用 realpath -m（需要 GNU coreutils）。
</platform_notes>

<success_criteria>
- [ ] 阶段在配置门检查之前验证
- [ ] 配置门已检查（workflow.code_review）
- [ ] 深度已解析并验证（quick|standard|deep）
- [ ] 文件范围通过三级计算：--files > SUMMARY.md > git diff
- [ ] 格式错误/缺失的 SUMMARY.md 通过回退优雅处理
- [ ] 已删除文件从范围中过滤掉
- [ ] 文件已去重和排序
- [ ] 空范围导致跳过（不启动 agent）
- [ ] Agent 已使用明确的文件列表和配置启动
- [ ] Agent 失败处理无部分提交
- [ ] REVIEW.md 如果已创建则已提交
- [ ] 结果以内联方式呈现并附下一步建议
</success_criteria>
