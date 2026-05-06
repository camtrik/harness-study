# 通用反模式

适用于所有工作流和 agent 的规则。个别工作流可能有额外的特定反模式。

---

## 上下文预算规则

1. **绝不**读取 agent 定义文件（`agents/*.md`）——`subagent_type` 会自动加载它们。将 agent 定义读入编排器会浪费用于自动注入子 agent 会话的上下文。
2. **绝不**将大文件内联到子 agent prompt 中——告诉 agent 从磁盘读取文件。Agent 有自己的上下文窗口。
3. **读取深度随上下文窗口伸缩**——检查 `.planning/config.json` 中的 `context_window`。在 < 500000 时：仅读取 frontmatter、状态字段或摘要。在 >= 500000（1M 模型）时：当内容需要内联决策时允许读取完整正文。参见 `references/context-budget.md` 获取完整表格。
4. **委托**繁重工作给子 agent——编排器负责路由，不构建、分析、研究、调查或验证。
5. **主动暂停警告**：如果你已经消耗了大量上下文（大文件读取、多个子 agent 结果），警告用户："上下文预算趋紧。建议检查点保存进度。"

## 文件读取规则

6. **SUMMARY.md 读取深度随上下文窗口伸缩**——在 context_window < 500000 时：仅读取先前阶段 SUMMARY 的 frontmatter。在 >= 500000 时：允许读取直接依赖阶段的完整正文。传递依赖（2+ 阶段之前）无论如何仅读取 frontmatter。
7. **绝不**从其他阶段读取完整的 PLAN.md 文件——仅读取当前阶段的计划。
8. **绝不**读取 `.planning/logs/` 文件——只有健康检查工作流读取这些。
9. **不要**在 frontmatter 足够时重新读取完整文件内容——frontmatter 包含 status、key_files、commits 和 provides 字段。例外：在 >= 500000 时，当需要语义内容时允许重新读取完整正文。

## 子 agent 规则

10. **绝对不要**使用非 GSD agent 类型（`general-purpose`、`Explore`、`Plan`、`Bash`、`feature-dev` 等）——始终使用 `subagent_type: "gsd-{agent}"`（例如 `gsd-phase-researcher`、`gsd-executor`、`gsd-planner`）。GSD agent 具有项目感知 prompt、审计日志和工作流上下文。通用 agent 绕过所有这些。
11. **不要**重新争论已在 CONTEXT.md（或 PROJECT.md `## Context` 节）中锁定的决策——无条件尊重锁定决策。

## 询问反模式

参考：`references/questioning.md` 获取完整的反模式列表。

12. **不要**走过清单——清单行走（从列表中逐一询问项目）是头号反模式。相反，使用渐进深度：从广泛开始，在有趣处深入。
13. **不要**使用企业用语——避免"利益相关者对齐"、"协同增效"、"可交付物"等行话。使用平实语言。
14. **不要**应用过早约束——在理解问题之前不要缩小解空间。先问问题，再约束。

## 状态管理反模式

15. **没有对 STATE.md 或 ROADMAP.md 的直接 Write/Edit 进行突变。** 始终使用 `gsd-sdk query` 进行已注册的状态/路线图处理（例如 `state.update`、`state.advance-plan`、`roadmap.update-plan-progress`），或旧版 `node …/gsd-tools.cjs` 进行仅 CLI 命令。直接 Write 工具使用绕过安全更新逻辑，在多会话环境中不安全。例外：允许从模板首次创建 STATE.md。

## 行为规则

16. **不要**创建用户未批准的产物——在写入新规划文档前始终确认。
17. **不要**修改工作流规定范围之外的文件——检查计划的 files_modified 列表。
18. **不要**在没有明确优先级的情况下建议多项下一步操作——一个主要建议，替代项列为次要。
19. **不要**使用 `git add .` 或 `git add -A`——仅暂存特定文件。
20. **不要**在规划文档或 commit 中包含敏感信息（API key、密码、token）。

## 错误恢复规则

21. **Git 锁检测**：在任何 git 操作之前，如果因"Unable to create lock file"失败，检查过时的 `.git/index.lock` 并建议用户删除（不要自动删除）。
22. **配置回退意识**：无效 JSON 时配置加载静默返回 `null`。如果你的工作流依赖配置值，检查 null 并警告用户："config.json 无效或缺失——以默认值运行。"
23. **部分状态恢复**：如果 STATE.md 引用不存在的阶段目录，不要静默继续。警告用户并建议诊断不匹配。

## GSD 特定规则

24. **不要**检查 `mode === 'auto'` 或 `mode === 'autonomous'`——GSD 使用 `yolo` 配置标志。检查 `yolo: true` 表示自主模式，缺失或 `false` 表示交互模式。
25. **当存在处理器时优先使用 `gsd-sdk query`** 进行编排；当用 shell 调用旧版 CLI 时，使用 **`gsd-tools.cjs`**（不是 `gsd-tools.js` 或任何其他文件名）——GSD 将程序化 API 发布为 CommonJS 以获得 Node.js CLI 兼容性。
26. **计划文件必须遵循 `{padded_phase}-{NN}-PLAN.md` 模式**（例如 `01-01-PLAN.md`）。绝不使用 `PLAN-01.md`、`plan-01.md` 或任何其他变体——gsd-tools 检测依赖于此精确模式。
27. **不要在写入当前计划的 SUMMARY.md 之前开始执行下一个计划**——下游计划可能通过 `@` 引用包含它。

## iOS / Apple 平台规则

28. **绝对不要使用 `Package.swift` + `.executableTarget`（或 `.target`）作为 iOS 应用的主构建系统。** SPM executable target 产生 macOS CLI 二进制，而非 iOS `.app` 包。它们无法安装在 iOS 设备上或提交到 App Store。使用 XcodeGen（`project.yml` + `xcodegen generate`）创建正确的 `.xcodeproj`。参见 `references/ios-scaffold.md` 获取完整模式。
29. **使用前验证 SwiftUI API 可用性。** 许多 SwiftUI API 需要特定的最低 iOS 版本（例如 `NavigationSplitView` 需要 iOS 16+，带多选的 `List(selection:)` 和 `@Observable` 需要 iOS 17）。如果计划使用的 API 超过声明的 `IPHONEOS_DEPLOYMENT_TARGET`，要么提高部署目标，要么添加 `#available` 守卫。
