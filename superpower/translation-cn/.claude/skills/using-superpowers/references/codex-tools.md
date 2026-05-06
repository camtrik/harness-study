# Codex 工具映射

Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些名称时，使用你平台的等价工具：

| Skill 中引用的工具 | Codex 等价工具 |
|-----------------|------------------|
| `Task` 工具（派发 subagent） | `spawn_agent`（请参阅[Subagent 派发需要多 agent 支持](#subagent-派发需要多-agent-支持)） |
| 多个 `Task` 调用（并行） | 多个 `spawn_agent` 调用 |
| Task 返回结果 | `wait_agent` |
| Task 自动完成 | `close_agent` 释放槽位 |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` 工具（调用 skill） | skills 原生加载——直接遵循指令 |
| `Read`、`Write`、`Edit`（文件） | 使用你的原生文件工具 |
| `Bash`（运行命令） | 使用你的原生 shell 工具 |

## Subagent 派发需要多 agent 支持

添加到你的 Codex 配置（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

这将启用 `spawn_agent`、`wait_agent` 和 `close_agent`，以支持诸如 `dispatching-parallel-agents` 和 `subagent-driven-development` 等 skill。

历史说明：`rust-v0.115.0` 之前的 Codex 版本将派发 agent 的等待功能暴露为 `wait`。当前的 Codex 使用 `wait_agent` 用于派发的 agent。`wait` 名称现在属于代码模式的 `exec/wait`，用于通过 `cell_id` 恢复一个已让出的 exec 单元；它不再是派发 agent 的结果工具。

## 环境检测

创建 worktree 或完成分支的 skills 应通过只读 git 命令在继续之前检测其环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已在链接的工作树中（跳过创建）
- `BRANCH` 为空 → 分离 HEAD（无法在沙箱中分支/推送/创建 PR）

请参阅 `using-git-worktrees` 第 0 步和 `finishing-a-development-branch` 第 1 步了解每个 skill 如何使用这些信号。

## Codex App 完成

当沙箱阻止分支/推送操作（在外部管理工作树中处于分离 HEAD 状态）时，agent 提交所有工作并告知用户使用 App 的原生控件：

- **"创建分支"**——命名分支，然后通过 App 界面提交/推送/创建 PR
- **"移交到本地"**——将工作转移到用户的本地检出

agent 仍然可以运行测试、暂存文件，并输出建议的分支名称、提交信息和 PR 描述供用户复制。
