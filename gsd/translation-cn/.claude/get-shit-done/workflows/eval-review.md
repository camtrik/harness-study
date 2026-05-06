<purpose>
对已实现 AI 阶段的评估覆盖范围进行回溯审计。独立的命令，适用于任何 GSD 管理的 AI 阶段。生成包含差距分析和修复计划的评分 EVAL-REVIEW.md。

在 /gsd-execute-phase 之后使用，以验证 AI-SPEC.md 中的评估策略是否已实际实施。镜像 /gsd-ui-review 和 /gsd-validate-phase 的模式。
</purpose>

<required_reading>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-evals.md
</required_reading>

<process>

## 0. 初始化

加载阶段操作上下文，解析模型，显示横幅。

## 1. 检测输入状态

检查 SUMMARY.md 和 AI-SPEC.md 是否存在：
- 状态 A：AI-SPEC.md + SUMMARY.md 存在 → 针对规范进行全面审计
- 状态 B：SUMMARY.md 存在，无 AI-SPEC.md → 针对通用最佳实践审计
- 状态 C：无 SUMMARY.md → 退出并报错

如果已存在 EVAL-REVIEW.md，询问是重新审计还是查看。

## 2. 收集上下文路径

构建审计器的文件列表（AI-SPEC.md、SUMMARY.md、PLAN.md）。

## 3. 启动 gsd-eval-auditor

使用正确的上下文启动评估审计器，传入 summary/plan/ai_spec 路径。

## 4. 解析审计器结果

读取生成的 EVAL-REVIEW.md，提取 overall_score、verdict 和 critical_gap_count。

## 5. 显示摘要

呈现格式化摘要，包含分数、裁决、关键差距数量以及基于裁决的适当下一步建议。

## 6. 提交

如果 commit_docs 为 true，提交 EVAL-REVIEW.md。

</process>

<success_criteria>
- [ ] 正确检测阶段执行状态
- [ ] 处理 AI-SPEC.md 存在/不存在的情况
- [ ] gsd-eval-auditor 使用正确的上下文启动
- [ ] EVAL-REVIEW.md 已写入（由审计器完成）
- [ ] 分数和裁决已向用户显示
- [ ] 根据裁决呈现适当的下一步
- [ ] 如果 commit_docs 启用则已提交
</success_criteria>
