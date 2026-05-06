# Agent 合约

所有 GSD agent 的完成标记和交接模式定义。工作流使用这些标记来检测 agent 完成情况并进行相应路由。

本文档描述实际状态而非理想状态。大小写不一致问题如实记录，与 agent 源文件中的表现一致。

---

## Agent 注册表

| Agent | 角色 | 完成标记 |
|-------|------|--------------------|
| gsd-planner | 计划创建 | `## PLANNING COMPLETE` |
| gsd-executor | 计划执行 | `## PLAN COMPLETE`、`## CHECKPOINT REACHED` |
| gsd-phase-researcher | 阶段范围研究 | `## RESEARCH COMPLETE`、`## RESEARCH BLOCKED` |
| gsd-project-researcher | 项目全局研究 | `## RESEARCH COMPLETE`、`## RESEARCH BLOCKED` |
| gsd-plan-checker | 计划验证 | `## VERIFICATION PASSED`、`## ISSUES FOUND` |
| gsd-research-synthesizer | 多研究综合 | `## SYNTHESIS COMPLETE`、`## SYNTHESIS BLOCKED` |
| gsd-debugger | 调试调查 | `## DEBUG COMPLETE`、`## ROOT CAUSE FOUND`、`## CHECKPOINT REACHED` |
| gsd-roadmapper | 路线图创建/修订 | `## ROADMAP CREATED`、`## ROADMAP REVISED`、`## ROADMAP BLOCKED` |
| gsd-ui-auditor | UI 审查 | `## UI REVIEW COMPLETE` |
| gsd-ui-checker | UI 验证 | `## ISSUES FOUND` |
| gsd-ui-researcher | UI 规格创建 | `## UI-SPEC COMPLETE`、`## UI-SPEC BLOCKED` |
| gsd-verifier | 执行后验证 | `## Verification Complete`（标题大小写） |
| gsd-integration-checker | 跨阶段集成检查 | `## Integration Check Complete`（标题大小写） |
| gsd-nyquist-auditor | 采样审计 | `## PARTIAL`、`## ESCALATE`（非标准） |
| gsd-security-auditor | 安全审计 | `## OPEN_THREATS`、`## ESCALATE`（非标准） |
| gsd-codebase-mapper | 代码库分析 | 无标记（直接写入文档） |
| gsd-assumptions-analyzer | 假设提取 | 无标记（返回 `## Assumptions` 节） |
| gsd-doc-verifier | 文档验证 | 无标记（向 `.planning/tmp/` 写入 JSON） |
| gsd-doc-writer | 文档生成 | 无标记（直接写入文档） |
| gsd-advisor-researcher | 咨询研究 | 无标记（工具型 agent） |
| gsd-user-profiler | 用户画像 | 无标记（在 analysis 标签中返回 JSON） |
| gsd-intel-updater | 代码库情报分析 | `## INTEL UPDATE COMPLETE`、`## INTEL UPDATE FAILED` |

## 标记规则

1. **全大写标记**（如 `## PLANNING COMPLETE`）是标准惯例
2. **标题大小写标记**（如 `## Verification Complete`）存在于 gsd-verifier 和 gsd-integration-checker —— 这是故意的设计，不是 bug
3. **非标准标记**（如 `## PARTIAL`、`## ESCALATE`）出现在审计 agent 中，表示需要编排器判断的部分结果
4. **无标记的 agent** 要么直接将产物写入磁盘，要么返回调用方解析的结构化数据（JSON/节）
5. 标记必须作为 H2 标题（`## `）出现在 agent 最终输出中的行首

## 关键交接合约

### Planner → Executor（通过 PLAN.md）

| 字段 | 必需 | 描述 |
|-------|------|-------------|
| Frontmatter | 是 | phase、plan、type、wave、depends_on、files_modified、autonomous、requirements |
| `<objective>` | 是 | 计划所要达成的目标 |
| `<tasks>` | 是 | 有序任务列表，包含 type、files、action、verify、acceptance_criteria |
| `<verification>` | 是 | 整体验证步骤 |
| `<success_criteria>` | 是 | 可衡量的完成标准 |

### Executor → Verifier（通过 SUMMARY.md）

| 字段 | 必需 | 描述 |
|-------|------|-------------|
| Frontmatter | 是 | phase、plan、subsystem、tags、key-files、metrics |
| Commits 表 | 是 | 每个任务的 commit 哈希和描述 |
| Deviations 节 | 是 | 自动修复的问题或 "None" |
| Self-Check | 是 | PASSED 或 FAILED 及详情 |

## 工作流正则匹配模式

工作流通过以下标记来检测 agent 完成：

**plan-phase.md 匹配：**
- `## RESEARCH COMPLETE` / `## RESEARCH BLOCKED`（researcher 输出）
- `## PLANNING COMPLETE`（planner 输出）
- `## CHECKPOINT REACHED`（planner/executor 暂停）
- `## VERIFICATION PASSED` / `## ISSUES FOUND`（plan-checker 输出）

**execute-phase.md 匹配：**
- `## PHASE COMPLETE`（阶段内所有计划完成）
- `## Self-Check: FAILED`（summary 自检）

> **注意：** `## PLAN COMPLETE` 是 gsd-executor 的完成标记，但 execute-phase.md 不会用正则匹配它。相反，它通过抽查（SUMMARY.md 是否存在、git commit 状态）来检测 executor 完成。这是有意为之，不是不匹配。
