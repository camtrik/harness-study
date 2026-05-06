<purpose>
在路线图中向当前里程碑的末尾添加一个新的整数编号阶段。自动计算下一个阶段编号，创建阶段目录，并更新路线图结构。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="parse_arguments">
解析命令参数：
- 所有参数构成阶段描述
- 示例：`/gsd-add-phase Add authentication` → 描述 = "Add authentication"
- 示例：`/gsd-add-phase Fix critical performance issues` → 描述 = "Fix critical performance issues"

如果未提供参数：

```
ERROR: Phase description required
Usage: /gsd-add-phase <description>
Example: /gsd-add-phase Add authentication system
```

退出。
</step>

<step name="init_context">
加载阶段操作上下文：

```bash
INIT=$(gsd-sdk query init.phase-op "0")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

检查 init JSON 中的 `roadmap_exists`。如果为 false：
```
ERROR: No roadmap found (.planning/ROADMAP.md)
Run /gsd-new-project to initialize.
```
退出。
</step>

<step name="add_phase">
**将阶段添加委托给 `gsd-sdk query phase.add`：**

```bash
RESULT=$(gsd-sdk query phase.add "${description}")
```

CLI 负责处理：
- 查找现有的最高整数阶段编号
- 计算下一个阶段编号（max + 1）
- 根据描述生成 slug
- 创建阶段目录（`.planning/phases/{NN}-{slug}/`）
- 将阶段条目插入 ROADMAP.md，包含目标、依赖关系和计划部分

从结果中提取：`phase_number`、`padded`、`name`、`slug`、`directory`。
</step>

<step name="update_project_state">
更新 STATE.md 以反映新阶段：

1. 读取 `.planning/STATE.md`
2. 在 "## Accumulated Context" → "### Roadmap Evolution" 下添加条目：
   ```
   - Phase {N} added: {description}
   ```

如果 "Roadmap Evolution" 部分不存在，则创建它。
</step>

<step name="completion">
呈现完成摘要：

```
Phase {N} added to current milestone:
- Description: {description}
- Directory: .planning/phases/{phase-num}-{slug}/
- Status: Not planned yet

Roadmap updated: .planning/ROADMAP.md

---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**Phase {N}: {description}**

`/clear` then:

`/gsd-plan-phase {N}`

---

**Also available:**
- `/gsd-add-phase <description>` — add another phase
- Review roadmap

---
```
</step>

</process>

<success_criteria>
- [ ] `gsd-sdk query phase.add` executed successfully
- [ ] Phase directory created
- [ ] Roadmap updated with new phase entry
- [ ] STATE.md updated with roadmap evolution note
- [ ] User informed of next steps
</success_criteria>
