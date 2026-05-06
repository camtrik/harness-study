# 每个计划的 worktree 决策（#2772）

在 `Task()` 分派之前为当前波次中的**每个计划**运行此逻辑。输出 `USE_WORKTREES_FOR_PLAN` 仅对该计划的调度分支进行门控 — 同一波次中的其他计划仍然可以走 worktree 路径。

`SUBMODULE_PATHS` 在 `initialize` 步骤中计算一次（从 `.gitmodules` 解析）。

`PLAN_FILES` 是从 `phase-plan-index` JSON 中提取的该计划声明将要触及的空格分隔路径列表。

运行每个计划的 gate：如果计划的路径与子模块路径有交集，则将此计划的 worktree 隔离设为 false。在 `execute_waves` 步骤 3 中的调度分支必须针对当前计划门控 `USE_WORKTREES_FOR_PLAN`，而不是项目级的 `USE_WORKTREES`。跟踪此波次中哪些计划实际使用了 worktrees（当 `USE_WORKTREES_FOR_PLAN != false` 时追加 plan_id 到 `WAVE_WORKTREE_PLANS` 累积器）。
