# 分块模式返回格式

当 `plan-phase` 以 `CHUNKED_MODE=true` 生成 `gsd-planner` 时使用（由 `--chunked` 标志或 `workflow.plan_chunked: true` 配置触发）。将单个长期运行的 planner Task 拆分为较短的 Task，以限制 Windows stdio 挂起的影响范围。

## 模式

### outline-only

**仅**写入 `{PHASE_DIR}/{PADDED_PHASE}-PLAN-OUTLINE.md`。不写入任何 PLAN.md 文件。
返回：

```markdown
## OUTLINE COMPLETE

**阶段：** {阶段名称}
**计划：** {N} 个计划，{M} 个 wave

| 计划 ID | 目标 | Wave | 依赖 | 需求 |
|---------|-----------|------|-----------|-------------|
| {padded_phase}-01 | [简要目标] | 1 | 无 | REQ-001, REQ-002 |
| {padded_phase}-02 | [简要目标] | 1 | 无 | REQ-003 |
```

编排器读取此表，然后为每一行生成一个单计划 Task。

### single-plan

写入**恰好一个** `{PHASE_DIR}/{plan_id}-PLAN.md`。不写入任何其他计划文件。
返回：

```markdown
## PLAN COMPLETE

**计划：** {plan-id}
**目标：** {简要}
**文件：** {PHASE_DIR}/{plan-id}-PLAN.md
**任务：** {N}
```

编排器在每次返回后验证文件在磁盘上存在，提交它，然后移至大纲中的下一个计划条目。

## 恢复行为

如果编排器检测到 `PLAN-OUTLINE.md` 已存在（来自之前中断的运行），则跳过 outline-only Task 直接进入 single-plan Task，跳过任何已在磁盘上存在的 `{plan_id}-PLAN.md` 文件。
