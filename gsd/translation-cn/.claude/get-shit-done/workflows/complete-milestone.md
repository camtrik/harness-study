<purpose>
将一个已发布的版本（v1.0、v1.1、v2.0）标记为完成。在 MILESTONES.md 中创建历史记录，执行完整的 PROJECT.md 演进审查，用里程碑分组重组 ROADMAP.md，并在 git 中标记发布。
</purpose>

<required_reading>
1. templates/milestone.md
2. templates/milestone-archive.md
3. `.planning/ROADMAP.md`
4. `.planning/REQUIREMENTS.md`
5. `.planning/PROJECT.md`
</required_reading>

<archival_behavior>
当里程碑完成时：
1. 提取完整里程碑详情到 `.planning/milestones/v[X.Y]-ROADMAP.md`
2. 归档需求到 `.planning/milestones/v[X.Y]-REQUIREMENTS.md`
3. 更新 ROADMAP.md（原地覆盖，保留 Backlog 部分）
4. 安全提交归档文件和更新后的 ROADMAP.md，然后 `git rm REQUIREMENTS.md`
5. 执行完整的 PROJECT.md 演进审查
6. 提供内联创建下一个里程碑的选项
7. 归档 UI 产物（*-UI-SPEC.md、*-UI-REVIEW.md）
8. 清理 `.planning/ui-reviews/` 截图文件（二进制资产，不归档）

**上下文效率：** 归档保持 ROADMAP.md 大小恒定，REQUIREMENTS.md 限制在里程碑范围内。
</archival_behavior>

<process>

<step name="pre_close_artifact_audit">
在继续里程碑关闭之前，运行全面的开放产物审计。如果存在开放项，显示审计报告并让用户选择解决、确认所有（推迟）或取消。如果确认，将确认项写入 STATE.md 的 `## Deferred Items` 部分。
</step>

<step name="verify_readiness">
使用 `roadmap analyze` 进行全面就绪检查，验证哪些阶段属于此里程碑、所有阶段是否完成以及进度百分比是否为 100%。解析 REQUIREMENTS.md 可追溯性表以检查需求完成情况。如果需求不完整，呈现继续/审计/中止选项。
</step>

<step name="gather_stats">
计算里程碑统计信息（阶段数、计划数、任务数、文件修改数、代码行数、时间线、git 范围）。
</step>

<step name="extract_accomplishments">
从 SUMMARY.md 文件中提取主要成就（4-6 个关键成就）。
</step>

<step name="create_milestone_entry">
MILESTONES.md 条目由 `gsd-sdk query milestone.complete` 自动创建。
</step>

<step name="evolve_project_full_review">
在里程碑完成时进行完整的 PROJECT.md 演进审查，包括："What This Is"准确性、核心价值检查、需求审计（已发布需求移至 Validated，添加新需求到 Active，审计 Out of Scope）、上下文更新、关键决策审计和约束检查。
</step>

<step name="reorganize_roadmap">
更新 `.planning/ROADMAP.md`，按里程碑分组完成阶段。先提取 Backlog 部分，再重写 ROADMAP.md，然后重新追加 Backlog 内容。
</step>

<step name="archive_milestone">
委托归档到 `gsd-sdk query milestone.complete`，CLI 处理创建归档、更新 STATE.md 等。AI 仍处理重组 ROADMAP.md、完整 PROJECT.md 演进审查、安全提交和 REQUIREMENTS.md 删除。
</step>

<step name="reorganize_roadmap_and_delete_originals">
归档后，重组 ROADMAP.md，先保存 Backlog 部分，然后重写。在删除原始文件之前提交归档文件作为安全检查点，然后通过 `git rm` 删除 REQUIREMENTS.md。
</step>

<step name="write_retrospective">
追加到活文档回顾中。从 SUMMARY.md、VERIFICATION.md、UAT.md 和 git 日志收集数据。编写里程碑部分（构建了什么、什么做得好、什么低效、建立的模式、关键教训、成本观察），并更新跨里程碑趋势。
</step>

<step name="update_state">
大部分 STATE.md 更新已由 `milestone complete` 处理，但验证和更新剩余字段（项目参考、累积上下文）。
</step>

<step name="handle_branches">
检查分支策略，提供合并选项（squash merge、带历史合并、不合并删除、保留分支），并处理相应操作。
</step>

<step name="git_tag">
创建 git tag 并可选推送到远程。
</step>

<step name="git_commit_milestone">
提交 REQUIREMENTS.md 删除。
</step>

<step name="offer_next">
呈现完成摘要和下一步建议（/gsd-new-milestone）。
</step>

</process>

<success_criteria>
- [ ] 收盘前产物审计已运行
- [ ] MILESTONES.md 条目已创建
- [ ] PROJECT.md 完整演进审查已完成
- [ ] ROADMAP.md 已以里程碑分组重组
- [ ] 路线图归档已创建
- [ ] 需求归档已创建
- [ ] 安全提交已在删除 REQUIREMENTS.md 之前完成
- [ ] REQUIREMENTS.md 已通过 git rm 删除
- [ ] STATE.md 已更新
- [ ] Git tag 已创建
- [ ] RETROSPECTIVE.md 已更新
- [ ] 用户知道下一步
</success_criteria>
