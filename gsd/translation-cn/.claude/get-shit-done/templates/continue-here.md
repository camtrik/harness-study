# Continue-Here 模板

复制并填充此结构到 `.planning/phases/XX-name/.continue-here.md`：

```yaml
---
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: 2025-01-15T14:30:00Z
---
```

```markdown
<current_state>
[我们具体在哪里？当前上下文是什么？]
</current_state>

<completed_work>
[本次会话完成了什么——请具体说明]

- 任务 1：[名称] - 已完成
- 任务 2：[名称] - 已完成
- 任务 3：[名称] - 进行中，[已完成的部分]
</completed_work>

<remaining_work>
[本阶段还剩什么]

- 任务 3：[名称] - [还需做什么]
- 任务 4：[名称] - 未开始
- 任务 5：[名称] - 未开始
</remaining_work>

<decisions_made>
[关键决策及其原因——以便下次会话无需重新讨论]

- 决定使用 [X]，因为 [原因]
- 选择了 [方法] 而非 [替代方案]，因为 [原因]
</decisions_made>

<blockers>
[任何卡住或等待外部因素的事项]

- [阻塞项 1]：[状态/临时方案]
</blockers>

<context>
[思维状态、"感觉"，任何有助于顺利恢复的内容]

[你在想什么？计划是什么？
这是“从你中断的地方无缝接续”的上下文。]
</context>

<next_action>
[恢复时要做的第一件事]

从以下开始：[具体操作]
</next_action>
```

<yaml_fields>
必需的 YAML frontmatter：

- `phase`：目录名（例如 `02-authentication`）
- `task`：当前任务编号
- `total_tasks`：阶段中的任务数量
- `status`：`in_progress`、`blocked`、`almost_done`
- `last_updated`：ISO 时间戳
</yaml_fields>

<guidelines>
- 要足够具体，使全新的 Claude 实例能立即理解
- 包含做出决策的原因，而不仅仅是决策内容
- `<next_action>` 应在无需阅读其他任何内容的情况下即可执行
- 此文件在恢复后会被删除——它不是永久存储
</guidelines>
