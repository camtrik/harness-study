# Gemini CLI 工具映射

Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些名称时，使用你平台的等价工具：

| Skill 中引用的工具 | Gemini CLI 等价工具 |
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
| `Task` 工具（派发 subagent） | `@agent-name`（请参阅[Subagent 支持](#subagent-支持)） |

## Subagent 支持

Gemini CLI 通过 `@` 语法原生支持 subagent。使用内置的 `@generalist` agent 来派发任何任务——它拥有所有工具的访问权限，并遵循你提供的 prompt。

当 skill 要求派发一个具名 agent 类型时，使用 `@generalist` 并附上 skill 的 prompt 模板中的完整 prompt：

| Skill 指令 | Gemini CLI 等价方式 |
|-------------------|----------------------|
| `Task 工具 (superpowers:implementer)` | `@generalist` 附上已填充的 `implementer-prompt.md` 模板 |
| `Task 工具 (superpowers:spec-reviewer)` | `@generalist` 附上已填充的 `spec-reviewer-prompt.md` 模板 |
| `Task 工具 (superpowers:code-reviewer)` | `@code-reviewer`（内置 agent）或 `@generalist` 附上已填充的审查 prompt |
| `Task 工具 (superpowers:code-quality-reviewer)` | `@generalist` 附上已填充的 `code-quality-reviewer-prompt.md` 模板 |
| `Task 工具 (general-purpose)` 附内联 prompt | `@generalist` 附上你的内联 prompt |

### Prompt 填充

Skills 提供了带有占位符的 prompt 模板，如 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]`。填充所有占位符，并将完整的 prompt 作为消息传递给 `@generalist`。prompt 模板本身包含了 agent 的角色、审查标准和期望的输出格式——`@generalist` 将遵循它。

### 并行派发

Gemini CLI 支持并行 subagent 派发。当 skill 要求你并行派发多个独立的 subagent 任务时，在同一条 prompt 中一起请求所有这些 `@generalist` 或具名 subagent 任务。有依赖关系的任务保持顺序执行，但不要仅仅为了保持更简单的历史记录而将独立的 subagent 任务串行化。

## 额外的 Gemini CLI 工具

这些工具在 Gemini CLI 中可用，但在 Claude Code 中没有等价工具：

| 工具 | 用途 |
|------|---------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将会话中的事实持久化到 GEMINI.md |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 丰富的任务管理（创建、更新、列表、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在修改之前切换到只读研究模式 |
