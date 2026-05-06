<purpose>
捕获在 GSD 会话中出现的一个想法、任务或问题，并将其结构化为一个待办事项，供后续处理。实现"想法 → 捕获 → 继续"的流程，而不会丢失上下文。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="init_context">
加载待办上下文：

```bash
INIT=$(gsd-sdk query init.todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

从 init JSON 中提取：`commit_docs`、`date`、`timestamp`、`todo_count`、`todos`、`pending_dir`、`todos_dir_exists`。

确保目录存在：
```bash
mkdir -p .planning/todos/pending .planning/todos/completed
```

记录 todos 数组中的现有区域，以便在 infer_area 步骤中保持一致性。
</step>

<step name="extract_content">
**有参数时：** 用作标题/主题。
- `/gsd-add-todo Add auth token refresh` → 标题 = "Add auth token refresh"

**无参数时：** 分析最近的对话以提取：
- 讨论的具体问题、想法或任务
- 提到的相关文件路径
- 技术细节（错误消息、行号、约束条件）

制定：
- `title`：3-10 词描述性标题（优先使用动作动词）
- `problem`：出了什么问题或为什么需要这个
- `solution`：方法提示或如果只是一个想法则为 "TBD"
- `files`：对话中提及的相关路径和行号
</step>

<step name="infer_area">
根据文件路径推断区域：

| 路径模式 | 区域 |
|--------------|------|
| `src/api/*`、`api/*` | `api` |
| `src/components/*`、`src/ui/*` | `ui` |
| `src/auth/*`、`auth/*` | `auth` |
| `src/db/*`、`database/*` | `database` |
| `tests/*`、`__tests__/*` | `testing` |
| `docs/*` | `docs` |
| `.planning/*` | `planning` |
| `scripts/*`、`bin/*` | `tooling` |
| 无文件或不明确 | `general` |

如果第 2 步中存在类似匹配，则使用现有区域。
</step>

<step name="check_duplicates">
```bash
# Search for key words from title in existing todos
grep -l -i "[key words from title]" .planning/todos/pending/*.md 2>/dev/null || true
```

如果发现潜在的重复项：
1. 读取现有待办
2. 比较范围


**文本模式（`workflow.text_mode: true` 在配置中或 `--text` 标志）：** 如果 `$ARGUMENTS` 中存在 `--text` 或 init JSON 中 `text_mode` 为 `true`，则设置 `TEXT_MODE=true`。当 TEXT_MODE 激活时，将每个 `AskUserQuestion` 调用替换为纯文本编号列表，并让用户输入其选择编号。这对于不支持 `AskUserQuestion` 的非 Claude 运行时（OpenAI Codex、Gemini CLI 等）是必需的。
如果重叠，使用 AskUserQuestion：
- header: "Duplicate?"
- question: "Similar todo exists: [title]. What would you like to do?"
- options:
  - "Skip" — 保留现有待办
  - "Replace" — 用新上下文更新现有待办
  - "Add anyway" — 创建为单独的待办
</step>

<step name="create_file">
使用 init 上下文中的值：`timestamp` 和 `date` 已可用。

为标题生成 slug：
```bash
slug=$(gsd-sdk query generate-slug "$title" --raw)
```

写入 `.planning/todos/pending/${date}-${slug}.md`：

```markdown
---
created: [timestamp]
title: [title]
area: [area]
files:
  - [file:lines]
---

## Problem

[problem description - enough context for future Claude to understand weeks later]

## Solution

[approach hints or "TBD"]
```
</step>

<step name="update_state">
如果 `.planning/STATE.md` 存在：

1. 使用 init 上下文中的 `todo_count`（如果计数更改则重新运行 `init todos`）
2. 更新 "## Accumulated Context" 下的 "### Pending Todos"
</step>

<step name="git_commit">
提交待办和任何更新的状态：

```bash
gsd-sdk query commit "docs: capture todo - [title]" --files .planning/todos/pending/[filename] .planning/STATE.md
```

该工具自动遵循 `commit_docs` 配置和 gitignore。

确认："Committed: docs: capture todo - [title]"
</step>

<step name="confirm">
```
Todo saved: .planning/todos/pending/[filename]

  [title]
  Area: [area]
  Files: [count] referenced

---

Would you like to:

1. Continue with current work
2. Add another todo
3. View all todos (/gsd-capture --list)
```
</step>

</process>

<success_criteria>
- [ ] Directory structure exists
- [ ] Todo file created with valid frontmatter
- [ ] Problem section has enough context for future Claude
- [ ] No duplicates (checked and resolved)
- [ ] Area consistent with existing todos
- [ ] STATE.md updated if exists
- [ ] Todo and state committed to git
</success_criteria>
