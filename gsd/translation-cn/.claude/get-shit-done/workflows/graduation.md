# LEARNINGS.md 跨阶段毕业助手

由 `transition.md` 步骤 `graduation_scan` 调用。从不为用户直接调用。

此工作流将最后 N 个阶段的 LEARNINGS.md 文件中重复出现的项目进行聚类，并通过 HITL 将升级候选呈现给开发者。任何项目都不会在没有明确开发者批准的情况下被升级。

配置（来自 config.json）：`features.graduation`（主开关）、`features.graduation_window`（扫描多少先前阶段，默认5）、`features.graduation_threshold`（最小聚类大小，默认3）。

流程：守卫检查→收集 LEARNINGS.md 文件→按词法相似性聚类（Jaccard 相似度）→检查 STATE.md 中的 graduation_backlog→呈现升级候选→HITL 处理每个聚类（Promote/Defer/Dismiss）→完成报告。
