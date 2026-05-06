<purpose>
将已完成里程碑中积累的阶段目录归档到 `.planning/milestones/v{X.Y}-phases/`。识别哪些阶段属于每个已完成的里程碑，显示试运行摘要，并在确认后移动目录。
</purpose>

<required_reading>
1. `.planning/MILESTONES.md`
2. `.planning/milestones/` 目录列表
3. `.planning/phases/` 目录列表
</required_reading>

<process>

<step name="identify_completed_milestones">
读取 `.planning/MILESTONES.md` 以识别已完成的里程碑及其版本。检查哪些里程碑归档目录已存在。过滤出尚未有 `-phases` 归档目录的里程碑。如果所有里程碑都已归档，显示消息并停止。
</step>

<step name="determine_phase_membership">
对于每个没有 `-phases` 归档的已完成里程碑，读取归档的 ROADMAP 快照以确定哪些阶段属于它。检查这些阶段目录中哪些仍然存在于 `.planning/phases/`。
</step>

<step name="show_dry_run">
为每个里程碑呈现试运行摘要，显示将归档哪些阶段目录以及目标路径。使用 AskUserQuestion 确认是否继续。
</step>

<step name="archive_phases">
对于每个里程碑，创建归档目录并将阶段目录移动到其中。

```bash
mkdir -p .planning/milestones/v{X.Y}-phases
mv .planning/phases/{dir} .planning/milestones/v{X.Y}-phases/
```
</step>

<step name="commit">
提交更改：

```bash
gsd-sdk query commit "chore: archive phase directories from completed milestones" --files .planning/milestones/ .planning/phases/
```
</step>

<step name="report">
显示每个里程碑归档的阶段数量和目标路径的摘要。
</step>

</process>

<success_criteria>
- [ ] 识别出所有没有现有阶段归档的已完成里程碑
- [ ] 从归档的 ROADMAP 快照确定阶段归属
- [ ] 显示试运行摘要并获得用户确认
- [ ] 阶段目录已移动到 `.planning/milestones/v{X.Y}-phases/`
- [ ] 更改已提交
</success_criteria>
