# Copilot CLI 工具映射

Skill 使用 Claude Code 工具名称。当你在 skill 中遇到这些工具时，使用你的平台等效项：

| Skill 引用 | Copilot CLI 等效项 |
|-----------------|----------------------|
| `Read`（文件读取） | `view` |
| `Write`（文件创建） | `create` |
| `Edit`（文件编辑） | `edit` |
| `Bash`（运行命令） | `bash` |
| `Grep`（搜索文件内容） | `grep` |
| `Glob`（按名称搜索文件） | `glob` |
| `Skill` 工具（调用 skill） | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（调度 subagent） | `task`，带有 `agent_type: "general-purpose"` 或 `"explore"` |
| 多个 `Task` 调用（并行） | 多个 `task` 调用 |
| Task 状态/输出 | `read_agent`、`list_agents` |
| `TodoWrite`（任务跟踪） | `sql`，带有内置的 `todos` 表 |
| `WebSearch` | 无等效项 - 使用 `web_fetch` 和搜索引擎 URL |
| `EnterPlanMode` / `ExitPlanMode` | 无等效项 - 保持在主会话中 |

## 异步 shell 会话

Copilot CLI 支持持久的异步 shell 会话，这没有直接的 Claude Code 等效项：

| 工具 | 目的 |
|------|---------|
| `bash`，带有 `async: true` | 在后台启动长时间运行的命令 |
| `write_bash` | 向正在运行的异步会话发送输入 |
| `read_bash` | 从异步会话读取输出 |
| `stop_bash` | 终止异步会话 |
| `list_bash` | 列出所有活动的 shell 会话 |

## 附加的 Copilot CLI 工具

| 工具 | 目的 |
|------|---------|
| `store_memory` | 持久化有关代码库的事实以供将来会话使用 |
| `report_intent` | 使用当前意图更新 UI 状态行 |
| `sql` | 查询会话的 SQLite 数据库（待办事项、元数据） |
| `fetch_copilot_cli_documentation` | 查阅 Copilot CLI 文档 |
| GitHub MCP 工具（`github-mcp-server-*`） | 原生 GitHub API 访问（问题、PR、代码搜索） |
