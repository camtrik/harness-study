<purpose>
通过汇总阶段验证、检查跨阶段集成和评估需求覆盖率，验证里程碑是否达到了其完成定义。读取现有的 VERIFICATION.md 文件，汇总技术债务和推迟的差距，然后启动集成检查器进行跨阶段连线检查。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<available_agent_types>
有效的 GSD subagent 类型（使用确切名称 — 不要回退到 'general-purpose'）：
- gsd-integration-checker — 检查跨阶段集成
</available_agent_types>

<process>

## 0. 初始化里程碑上下文

```bash
INIT=$(gsd-sdk query init.milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_CHECKER=$(gsd-sdk query agent-skills gsd-integration-checker)
```

从 init JSON 提取里程碑版本、里程碑名称、阶段数量、已完成阶段和提交文档配置。

## 1. 确定里程碑范围

获取里程碑中的阶段并解析版本。识别范围内的所有阶段目录，提取里程碑的完成定义和映射到此里程碑的需求。

## 2. 读取所有阶段验证

对于每个阶段目录，读取 VERIFICATION.md，提取：
- 状态（passed/gaps_found）
- 关键差距（如果有 — 这些是阻塞项）
- 非关键差距（技术债务、推迟项、警告）
- 发现的反模式（TODOs、stub、占位符）
- 需求覆盖率（哪些需求已满足/被阻塞）

如果阶段缺少 VERIFICATION.md，标记为"未验证阶段" — 这是一个阻塞项。

## 3. 启动集成检查器

收集阶段上下文后，提取里程碑需求 ID 并以 phase 目录和 API 路由为输入启动 gsd-integration-checker subagent。

> **编排器规则 — CODEX 运行时**：调用 Task() 后，立即停止此任务的工作。在 subagent 活动期间，不要读取更多文件、编辑代码或运行与此任务相关的测试。等待 subagent 返回结果。

## 4. 收集结果

合并阶段级差距和技术债务以及集成检查器的报告。

## 5. 需求覆盖率检查（三源交叉引用）

必须对每个需求交叉引用三个独立来源：
- 解析 REQUIREMENTS.md 可追溯性表
- 解析每个阶段 VERIFICATION.md 的需求表
- 提取 SUMMARY.md frontmatter 中的 `requirements-completed`

根据三个来源的状态矩阵（VERIFICATION.md 状态 × SUMMARY Frontmatter × REQUIREMENTS.md 勾选状态）确定最终状态（satisfied/partial/unsatisfied）。任何 unsatisfied 需求必须强制 milestones 审计为 gaps_found 状态。还要检测孤儿需求。

## 5.5. Nyquist 合规性发现

如果 `workflow.nyquist_validation` 未明确设为 false，则扫描每个阶段的 VALIDATION.md 来评估合规性。仅发现，不自动调用 `/gsd-validate-phase`。

## 6. 汇总到里程碑审计文件

创建 `.planning/v{version}-v{version}-MILESTONE-AUDIT.md`，包含结构化的 YAML frontmatter（里程碑信息、状态、分数、差距和技术债务）以及完整的 markdown 报告。

## 7. 呈现结果

根据状态路由输出（passed → 完成里程碑建议，gaps_found → 差距修复建议，tech_debt → 债务审查选项）。

</process>

<success_criteria>
- [ ] Milestone scope identified
- [ ] All phase VERIFICATION.md files read
- [ ] 3-source cross-reference completed
- [ ] Orphaned requirements detected
- [ ] Tech debt and deferred gaps aggregated
- [ ] Integration checker spawned
- [ ] MILESTONE-AUDIT.md created
- [ ] FAIL gate enforced
- [ ] Nyquist compliance scanned (if enabled)
- [ ] Results presented with actionable next steps
</success_criteria>
