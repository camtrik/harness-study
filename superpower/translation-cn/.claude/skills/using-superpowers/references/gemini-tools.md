# Gemini CLI 工具映射

Skill 使用 Claude Code 工具名称。当你在 skill 中遇到这些工具时，使用你的平台等效项：

| Skill 引用 | Gemini CLI 等效项 |
|-----------------|----------------------|
| `Read`（文件读取） | `read_file` |
| `Write`（文件创建） | `write_file` |
| `Edit`（文件编辑） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名称搜索文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` 工具（调用 skill） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（调度 subagent） | `@agent-name`（参见 [Subagent 支持](#subagent-support)） |

## Subagent 支持

Gemini CLI 通过 `@` 语法原生支持 subagent。使用内置的 `@generalist` agent 调度任何任务 - 它有权访问所有工具并遵循你提供的提示。

当 skill 说要调度命名的 agent 类型时，使用 `@generalist` 和 skill 提示模板中的完整提示：

| Skill 指令 | Gemini CLI 等效项 |
|-------------------|----------------------|
| `Task 工具（superpowers:implementer）` | `@generalist`，带有填充的 `implementer-prompt.md` 模板 |
| `Task 工具（superpowers:spec-reviewer）` | `@generalist`，带有填充的 `spec-reviewer-prompt.md` 模板 |
| `Task 工具（superpowers:code-reviewer）` | `@code-reviewer`（捆绑的 agent）或 `@generalist`，带有填充的审查提示 |
| `Task 工具（superpowers:code-quality-reviewer）` | `@generalist`，带有填充的 `code-quality-reviewer-prompt.md` 模板 |
| `Task 工具（general-purpose）`，带有内联提示 | `@generalist`，带有你的内联提示 |

### 提示填充

Skill 提供带有占位符的提示模板，如 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]`。填充所有占位符并将完整提示作为消息传递给 `@generalist`。提示模板本身包含 agent 的角色、审查标准和预期的输出格式 - `@generalist` 将遵循它。

### 并行调度

Gemini CLI 支持并行 subagent 调度。当 skill 要求你在并行中调度多个独立的 subagent 任务时，在同一提示中一起请求所有这些 `@generalist` 或命名的 subagent 任务。保持依赖任务的顺序，但不要为了保留更简单的历史记录而序列化独立的 subagent 任务。

## 附加的 Gemini CLI 工具

这些工具在 Gemini CLI 中可用，但没有 Claude Code 等效项：

| 工具 | 目的 |
|------|---------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将事实持久化到 GEMINI.md 跨会话 |
| `ask_user` | 请求用户的结构化输入 |
| `tracker_create_task` | 丰富的任务管理（创建、更新、列表、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在进行更改之前切换到只读研究模式 |
