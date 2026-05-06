# CONTEXT.md 模板 — 用于讨论阶段 write_context 步骤

> **延迟加载。** 仅在 `workflows/discuss-phase.md` 的 `write_context` 步骤内读取此文件，紧接在写入 `${phase_dir}/${padded_phase}-CONTEXT.md` 之前。

## 变量替换

调用者替换：
- `[X]` → 阶段编号
- `[Name]` → 阶段名称
- `[date]` → 收集上下文的 ISO 日期
- `${padded_phase}` → 零填充的阶段编号（例如 `07`、`15`）
- `{N}` → 计数（需求数量等）

## 条件部分

- **`<spec_lock>`** — 仅当 `spec_loaded = true` 时包含（`check_spec` 找到 `*-SPEC.md`）。否则完全省略。
- **Folded Todos / Reviewed Todos** — 仅当 `cross_reference_todos` 步骤折叠或审查了至少一个待办时才包含子部分。

## 模板正文

```markdown
# Phase [X]: [Name] - Context

**Gathered:** [date]
**Status:** Ready for planning

<domain>
## Phase Boundary

[此阶段交付内容的明确陈述 — 范围锚点]

</domain>

[如果 spec_loaded = true，插入此部分：]
<spec_lock>
## Requirements (locked via SPEC.md)

**{N} 个需求已锁定。** 参见 `{padded_phase}-SPEC.md` 获取完整的需求、边界和验收标准。

下游 agent 在规划或实现之前必须阅读 `{padded_phase}-SPEC.md`。需求不在此处重复。

</spec_lock>

<decisions>
## Implementation Decisions

### [讨论过的类别 1]
- **D-01:** [捕获的决策或偏好]
- **D-02:** [另一个决策（如适用）]

### [讨论过的类别 2]
- **D-03:** [捕获的决策或偏好]

### Claude's Discretion
[用户说"you decide"的领域 — 注意 Claude 在此有灵活性的地方]

### Folded Todos
[如果从 cross_reference_todos 步骤有折叠的待办，在此列出]

</decisions>

<canonical_refs>
## Canonical References

**下游 agent 在规划或实现之前必须阅读这些。**

[强制部分。按主题区域分组，每个条目需要完整的相对路径。]

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- [组件/hook/工具]：[如何在此阶段使用]

### Established Patterns
- [模式]：[如何约束/启用此阶段]

### Integration Points
- [新代码与现有系统的连接点]

</code_context>

<specifics>
## Specific Ideas

[来自讨论的任何特定参考、示例或"我想要像 X 那样的"时刻]

</specifics>

<deferred>
## Deferred Ideas

[出现了但属于其他阶段的想法。不要丢失它们。]

</deferred>

---

*Phase: [X]-[Name]*
*Context gathered: [date]*
```
