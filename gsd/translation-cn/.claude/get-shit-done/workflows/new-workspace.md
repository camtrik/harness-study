<purpose>
创建一个隔离的工作区目录，包含 git 仓库副本（worktree 或 clone）和独立的 `.planning/` 目录。支持多仓库编排和单仓库特性分支隔离。
</purpose>
<process>
解析参数（--name 必填、--repos、--path、--strategy、--branch）。选择仓库和策略（worktree/clone）。验证目标路径和源仓库。创建 worktree/clone。写入 WORKSPACE.md 清单。初始化 .planning/。报告并提供初始化 GSD 的选项。
</process>
