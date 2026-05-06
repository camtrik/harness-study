# Planner Subagent Prompt 模板

用于生成 gsd-planner agent 的模板。agent 包含了所有规划专业知识——此模板仅提供规划上下文。

---

## 模板

```markdown
<planning_context>

**阶段：** {phase_number}
**模式：** {standard | gap_closure}

**项目状态：**
@.planning/STATE.md

**路线图：**
@.planning/ROADMAP.md

**需求（如存在）：**
@.planning/REQUIREMENTS.md

**阶段上下文（如存在）：**
@.planning/phases/{phase_dir}/{phase_num}-CONTEXT.md

**研究（如存在）：**
@.planning/phases/{phase_dir}/{phase_num}-RESEARCH.md

**缺口修复（如果使用 --gaps 模式）：**
@.planning/phases/{phase_dir}/{phase_num}-VERIFICATION.md
@.planning/phases/{phase_dir}/{phase_num}-UAT.md

</planning_context>

<downstream_consumer>
输出由 /gsd-execute-phase 消费
计划必须是可执行的 prompt，包含：
- Frontmatter（wave、depends_on、files_modified、autonomous）
- XML 格式的任务
- 验证标准
- 用于目标回溯验证的 must_haves
</downstream_consumer>

<quality_gate>
在返回 PLANNING COMPLETE 之前：
- [ ] PLAN.md 文件在阶段目录中创建
- [ ] 每个计划都有有效的 frontmatter
- [ ] 任务是具体且可执行的
- [ ] 依赖关系已正确识别
- [ ] 波次已为并行执行分配
- [ ] must_haves 已从阶段目标推导得出
</quality_gate>
```

---

## 占位符

| 占位符 | 来源 | 示例 |
|-------------|--------|---------|
| `{phase_number}` | 来自路线图/参数 | `5` 或 `2.1` |
| `{phase_dir}` | 阶段目录名 | `05-user-profiles` |
| `{phase}` | 阶段前缀 | `05` |
| `{standard \| gap_closure}` | 模式标志 | `standard` |

---

## 用法

**来自 /gsd-plan-phase（标准模式）：**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-planner",
  description="规划阶段 {phase}"
)
```

**来自 /gsd-plan-phase --gaps（缺口修复模式）：**
```python
Task(
  prompt=filled_template,  # 使用 mode: gap_closure
  subagent_type="gsd-planner",
  description="规划阶段 {phase} 的缺口修复"
)
```

---

## 继续规划

对于检查点，使用以下内容生成新的 agent：

```markdown
<objective>
继续规划阶段 {phase_number}：{phase_name}
</objective>

<prior_state>
阶段目录：@.planning/phases/{phase_dir}/
已有计划：@.planning/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**类型：** {checkpoint_type}
**回复：** {user_response}
</checkpoint_response>

<mode>
继续：{standard | gap_closure}
</mode>
```

---

**注意：** 规划方法论、任务分解、依赖分析、波次分配、TDD 检测和目标回溯推导已内置于 gsd-planner agent 中。此模板仅传递上下文。
