# --text 模式 — 纯文本叠加（不使用 AskUserQuestion）

> **延迟加载叠加。** 当 `$ARGUMENTS` 中存在 `--text`，或配置中设置了 `workflow.text_mode: true` 时，从 `workflows/discuss-phase.md` 读取此文件。

## 效果

当文本模式激活时，**完全不使用 AskUserQuestion**。取而代之，将每个问题呈现为纯文本编号列表，并让用户输入其选择编号。自由文本输入映射到等效 AskUserQuestion 调用的 "Other" 分支。

这对于 Claude Code 远程会话（`/rc` 模式）是必需的，因为 Claude App 无法将 TUI 菜单选择转发回主机。

## 激活

- 按会话：将 `--text` 标志传递给任何命令（例如 `/gsd-discuss-phase --text`）
- 按项目：`gsd-sdk query config-set workflow.text_mode true`

文本模式适用于会话中的所有工作流，不仅仅是讨论阶段。

## 问题渲染

将 AskUserQuestion 调用替换为纯文本编号列表。等待用户在正常提示符下回复。解析：
- 数字回复 → 映射到该选项
- 自由文本 → 视为"Other" — 反映回来，确认，然后继续

## 空答案处理

适用与父文件相同的答案验证规则：空响应触发一次重试，然后是澄清问题。不要以空输入继续。
