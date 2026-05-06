---
name: gsd:update
description: 将 GSD 更新到最新版本并显示更新日志
argument-hint: "[--sync | --reapply]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
检查 GSD 更新，如有可用更新则安装，并显示变更内容。

路由到 update 工作流，该工作流处理：
- 版本检测（本地 vs 全局安装）
- npm 版本检查
- 更新日志获取和显示
- 用户确认（含全新安装警告）
- 更新执行和缓存清除
- 重启提醒
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/update.md
</execution_context>

<flags>
- **--sync**：跨运行时根目录同步托管的 GSD skills，使多运行时用户在更新后保持对齐。运行 sync-skills 工作流（支持 --from、--to、--dry-run、--apply 标志）。
- **--reapply**：在 GSD 更新后重新应用本地修改。使用三方比较（原始基线、用户修改备份、新安装版本）将用户自定义内容合并回去。运行 reapply-patches 工作流。
- **（无标志）**：标准更新 — 检查新版本、显示更新日志、安装。
</flags>

<process>
解析 $ARGUMENTS 的第一个 token：
- 如果是 `--sync`：去除该标志，执行 sync-skills 工作流（传递剩余的 --from/--to/--dry-run/--apply 参数）。
- 如果是 `--reapply`：去除该标志，执行 reapply-patches 工作流。
- 否则：**遵循 update 工作流**，来自 `@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/update.md`。

update 工作流处理所有逻辑，包括：
1. 已安装版本检测（本地/全局）
2. 通过 npm 检查最新版本
3. 版本比较
4. 更新日志获取和提取
5. 全新安装警告显示
6. 用户确认
7. 更新执行
8. 缓存清除
</process>

<execution_context_extended>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/sync-skills.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/reapply-patches.md
</execution_context_extended>
