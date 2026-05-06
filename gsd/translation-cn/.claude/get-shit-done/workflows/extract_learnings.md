<purpose>
从已完成的阶段产物（PLAN.md、SUMMARY.md、VERIFICATION.md、UAT.md、STATE.md）中提取决策、经验教训、发现的模式和意外情况，并结构化为 LEARNINGS.md 文件。捕获可能在各阶段间丢失的机构知识。
</purpose>

<process>

<step name="initialize">
加载阶段操作上下文。如果未找到阶段，退出并报错。
</step>

<step name="collect_artifacts">
读取阶段产物。PLAN.md 和 SUMMARY.md 是必需的；VERIFICATION.md、UAT.md 和 STATE.md 是可选的。跟踪缺失的可选产物。
</step>

<step name="extract_learnings">
将所有收集到的产物中的学习提取到 4 个类别：
1. **决策** — 技术和架构决策（What/Why/Source）
2. **经验教训** — 执行中了解到的事先未知的东西（What/Context/Source）
3. **模式** — 可重用的模式、方法或技术（Pattern/When to use/Source）
4. **意外** — 意外的发现、行为或结果（What/Impact/Source）

每个条目必须包含来源归属。
</step>

<step name="write_learnings">
将 LEARNINGS.md 写入阶段目录，包含 YAML frontmatter（phase、phase_name、project、counts、missing_artifacts）。
</step>

<step name="report">
呈现提取摘要并显示计数。
</step>

</process>

<critical_rules>
- PLAN.md 和 SUMMARY.md 是必需的
- VERIFICATION.md、UAT.md 和 STATE.md 是可选的
- 每个提取的学习必须有来源归属
- 运行 extract-learnings 两次必须覆盖（替换）而非追加
- 不要编造学习 — 只提取产物中明确记录的内容
</critical_rules>

<success_criteria>
- [ ] 阶段产物已定位并成功读取
- [ ] 所有 4 个类别已提取
- [ ] 每个条目有来源归属
- [ ] LEARNINGS.md 已使用正确的 YAML frontmatter 写入
- [ ] STATE.md 已更新提取活动
- [ ] 用户收到摘要报告
</success_criteria>
