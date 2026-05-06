<purpose>
为涉及构建 AI 系统的阶段生成 AI 设计契约（AI-SPEC.md）。编排 gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner，并附有验证门。在 GSD 生命周期中插入到 discuss-phase 和 plan-phase 之间。

AI-SPEC.md 在规划器创建任务之前锁定四件事：
1. 框架选择（附理由和替代方案）
2. 实现指导（来自官方文档的正确语法、模式和陷阱）
3. 领域上下文（实践者评分标准要素、故障模式、监管约束）
4. 评估策略（维度、评分标准、工具、参考数据集、防护栏）

这可以防止 AI 开发中最常见的两种失败：为用例选择错误的框架，以及将评估视为事后补充。
</purpose>

<required_reading>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-frameworks.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ai-evals.md
</required_reading>

<process>

## 1. Initialize

```bash
INIT=$(gsd-sdk query init.plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

解析 JSON 获取：`phase_dir`、`phase_number`、`phase_name`、`phase_slug`、`padded_phase`、`has_context`、`has_research`、`commit_docs`。

**文件路径：** `state_path`、`roadmap_path`、`requirements_path`、`context_path`。

解析 agent 模型：
```bash
SELECTOR_MODEL=$(gsd-sdk query resolve-model gsd-framework-selector 2>/dev/null | jq -r '.model' 2>/dev/null || true)
RESEARCHER_MODEL=$(gsd-sdk query resolve-model gsd-ai-researcher 2>/dev/null | jq -r '.model' 2>/dev/null || true)
DOMAIN_MODEL=$(gsd-sdk query resolve-model gsd-domain-researcher 2>/dev/null | jq -r '.model' 2>/dev/null || true)
PLANNER_MODEL=$(gsd-sdk query resolve-model gsd-eval-planner 2>/dev/null | jq -r '.model' 2>/dev/null || true)
```

检查配置：
```bash
AI_PHASE_ENABLED=$(gsd-sdk query config-get workflow.ai_integration_phase 2>/dev/null || echo "true")
```

**如果 `AI_PHASE_ENABLED` 为 `false`：**
```
AI phase is disabled in config. Enable via /gsd-settings.
```
退出工作流。

**如果 `planning_exists` 为 false：** 错误 — 先运行 `/gsd-new-project`。

## 2. 解析并验证阶段

从 $ARGUMENTS 中提取阶段编号。如果未提供，检测下一个未规划的阶段。

```bash
PHASE_INFO=$(gsd-sdk query roadmap.get-phase "${PHASE}")
```

**如果 `found` 为 false：** 报错并显示可用阶段列表。

## 3. 检查先决条件

**如果 `has_context` 为 false：**
```
No CONTEXT.md found for Phase {N}.
Recommended: run /gsd-discuss-phase {N} first to capture framework preferences.
Continuing without user decisions — framework selector will ask all questions.
```
继续（非阻塞）。

## 4. 检查现有 AI-SPEC

```bash
AI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-AI-SPEC.md 2>/dev/null | head -1)
```

**文本模式（`workflow.text_mode: true` 在配置中或 `--text` 标志）：** 如果 `$ARGUMENTS` 中存在 `--text` 或 init JSON 中 `text_mode` 为 `true`，则设置 `TEXT_MODE=true`。当 TEXT_MODE 激活时，将每个 `AskUserQuestion` 调用替换为纯文本编号列表，并让用户输入其选择编号。

**如果存在：** 使用 AskUserQuestion：
- header: "Existing AI-SPEC"
- question: "AI-SPEC.md already exists for Phase {N}. What would you like to do?"
- options:
  - "Update — re-run with existing as baseline"
  - "View — display current AI-SPEC and exit"
  - "Skip — keep current AI-SPEC and exit"

如果 "View"：显示文件内容，退出。
如果 "Skip"：退出。
如果 "Update"：继续执行步骤 5。

## 5. 启动 gsd-framework-selector

显示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AI DESIGN CONTRACT — PHASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Step 1/4 — Framework Selection...
```

启动 `gsd-framework-selector`，传入：
```markdown
Read /Users/ebbi/Work/harness-study/gsd/.claude/agents/gsd-framework-selector.md for instructions.

<objective>
Select the right AI framework for Phase {phase_number}: {phase_name}
Goal: {phase_goal}
</objective>

<files_to_read>
{context_path if exists}
{requirements_path if exists}
</files_to_read>

<phase_context>
Phase: {phase_number} — {phase_name}
Goal: {phase_goal}
</phase_context>
```

解析选择器输出获取：`primary_framework`、`system_type`、`model_provider`、`eval_concerns`、`alternative_framework`。

**如果选择器失败或返回空：** 报错退出 — "Framework selection failed. Re-run /gsd-ai-integration-phase {N} or answer the framework question in /gsd-discuss-phase {N} first."

## 6. 初始化 AI-SPEC.md

复制模板：
```bash
cp "/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/AI-SPEC.md" "${PHASE_DIR}/${PADDED_PHASE}-AI-SPEC.md"
```

填充头字段：
- 阶段编号和名称
- 系统分类（来自选择器）
- 选定框架（来自选择器）
- 考虑的替代方案（来自选择器）

## 7. 启动 gsd-ai-researcher

显示：
```
◆ Step 2/4 — Researching {primary_framework} docs + AI systems best practices...
```

启动 `gsd-ai-researcher`（编写 AI-SPEC.md 的第 3 和第 4 节）。

## 8. 启动 gsd-domain-researcher

显示：
```
◆ Step 3/4 — Researching domain context and expert evaluation criteria...
```

启动 `gsd-domain-researcher`（编写 AI-SPEC.md 的第 1b 节领域上下文）。

## 9. 启动 gsd-eval-planner

显示：
```
◆ Step 4/4 — Designing evaluation strategy from domain + technical context...
```

启动 `gsd-eval-planner`（编写 AI-SPEC.md 的第 5、6、7 节评估策略）。

## 10. 验证 AI-SPEC 完整性

读取完成的 AI-SPEC.md。检查：
- 第 2 节有框架名称（不是占位符）
- 第 1b 节至少有一个领域评分标准要素（Good/Bad/Stakes）
- 第 3 节有一个非空的代码块（入口点模式）
- 第 4b 节有 Pydantic 示例
- 第 5 节维度表中至少有一行
- 第 6 节至少有一个防护栏或明确的"N/A for internal tool"说明
- 末尾核对清单部分有 3 个以上已勾选项

**如果验证失败：** 显示缺失的具体部分。询问用户是否要重新运行特定步骤还是继续。

## 11. 提交

**如果 `commit_docs` 为 true：**
```bash
git add "${AI_SPEC_FILE}"
git commit -m "docs({phase_slug}): generate AI-SPEC.md — {primary_framework} + domain context + eval strategy"
```

## 12. 显示完成信息

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AI-SPEC COMPLETE — PHASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Framework: {primary_framework}
◆ System Type: {system_type}
◆ Domain: {domain_vertical from Section 1b}
◆ Eval Dimensions: {eval_concerns}
◆ Tracing Default: Arize Phoenix (or detected existing tool)
◆ Output: {ai_spec_path}

Next step:
  /gsd-plan-phase {N}   — planner will consume AI-SPEC.md
```

</process>

<success_criteria>
- [ ] Framework selected with rationale (Section 2)
- [ ] AI-SPEC.md created from template
- [ ] Framework docs + AI best practices researched (Sections 3, 4, 4b populated)
- [ ] Domain context + expert rubric ingredients researched (Section 1b populated)
- [ ] Eval strategy grounded in domain context (Sections 5-7 populated)
- [ ] Arize Phoenix (or detected tool) set as tracing default in Section 7
- [ ] AI-SPEC.md validated (Sections 1b, 2, 3, 4b, 5, 6 all non-empty)
- [ ] Committed if commit_docs enabled
- [ ] Next step surfaced to user
</success_criteria>
