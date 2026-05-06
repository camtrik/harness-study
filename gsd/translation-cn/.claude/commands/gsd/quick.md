---
name: gsd:quick
description: 快速任务执行，具备 GSD 保障（原子提交、状态跟踪），但跳过可选 agent
argument-hint: "[list | status <slug> | resume <slug> | --full] [--validate] [--discuss] [--research] [task description]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
执行小型的临时任务，具备 GSD 保障（原子提交、STATE.md 跟踪）。

Quick 模式是相同的系统，只是路径更短：
- 启动 gsd-planner（quick 模式）+ gsd-executor(s)
- Quick 任务驻留在 `.planning/quick/` 中，与计划阶段分离
- 更新 STATE.md "Quick Tasks Completed" 表格（而非 ROADMAP.md）

**默认：** 跳过研究、讨论、plan-checker、verifier。当你确切知道要做什么时使用。

**`--discuss` 标志：** 规划前的轻量级讨论。暴露假设、澄清灰区、在 CONTEXT.md 中捕获决策。当任务有值得预先解决的歧义时使用。

**`--full` 标志：** 启用完整的质量流水线 — 讨论 + 研究 + 计划检查 + 验证。一个标志启用所有功能。

**`--validate` 标志：** 仅启用计划检查（最多 2 次迭代）和执行后验证。当你需要质量保障但不需要讨论或研究时使用。

**`--research` 标志：** 规划前启动一个专注的研究 agent。调查任务的可能实现方案、库选项和陷阱。当你不确定最佳实现方法时使用。

细粒度标志可组合使用：`--discuss --research --validate` 与 `--full` 效果相同。

**子命令：**
- `list` — 列出所有 quick 任务及其状态
- `status <slug>` — 显示特定 quick 任务的状态
- `resume <slug>` — 通过 slug 恢复特定 quick 任务
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/quick.md
</execution_context>

<context>
$ARGUMENTS

上下文文件在工作流中解析（`init quick`），并通过 `<files_to_read>` 块委托。
</context>

<process>

**首先解析 $ARGUMENTS 以获取子命令：**

- 如果 $ARGUMENTS 以 "list" 开头：SUBCMD=list
- 如果 $ARGUMENTS 以 "status " 开头：SUBCMD=status，SLUG=剩余部分（去除空格，净化）
- 如果 $ARGUMENTS 以 "resume " 开头：SUBCMD=resume，SLUG=剩余部分（去除空格，净化）
- 否则：SUBCMD=run，将完整的 $ARGUMENTS 原样传递给 quick 工作流

**Slug 净化（用于 status 和 resume）：** 去除任何不匹配 `[a-z0-9-]` 的字符。拒绝长度超过 60 个字符或包含 `..` 或 `/` 的 slug。如果无效，输出 "无效的会话 slug。" 并停止。

## LIST 子命令

当 SUBCMD=list 时：

```bash
ls -d .planning/quick/*/  2>/dev/null
```

对于找到的每个目录：
- 检查 PLAN.md 是否存在
- 检查 SUMMARY.md 是否存在；如果存在，通过以下方式从 frontmatter 读取 `status`：
  ```bash
  gsd-sdk query frontmatter.get .planning/quick/{dir}/SUMMARY.md status
  ```
- 确定目录创建日期：`stat -f "%SB" -t "%Y-%m-%d"`（macOS）或 `stat -c "%w"`（Linux）；回退到目录名称中的日期前缀（格式：`YYYYMMDD-` 前缀）
- 推导显示状态：
  - SUMMARY.md 存在，frontmatter status=complete → `complete ✓`
  - SUMMARY.md 存在，frontmatter status=incomplete 或 status 缺失 → `incomplete`
  - SUMMARY.md 缺失，目录创建于 <7 天前 → `in-progress`
  - SUMMARY.md 缺失，目录创建于 ≥7 天前 → `abandoned? (>7 days, no summary)`

**安全声明：** 目录名从文件系统读取。在显示任何 slug 之前，进行净化：使用 `name.replace(/[^\x20-\x7E]/g, '').replace(/[/\\]/g, '')` 去除不可打印字符、ANSI 转义序列和路径分隔符。切勿将原始目录名直接用于 shell 命令的字符串插值。

显示格式：
```
Quick Tasks
────────────────────────────────────────────────────────────
slug                           date        status
backup-s3-policy               2026-04-10  in-progress
auth-token-refresh-fix         2026-04-09  complete ✓
update-node-deps               2026-04-08  abandoned? (>7 days, no summary)
────────────────────────────────────────────────────────────
3 tasks (1 complete, 2 incomplete/in-progress)
```

如果未找到目录：打印 `未找到 Quick 任务。` 并停止。

显示列表后停止。不要继续执行后续步骤。

## STATUS 子命令

当 SUBCMD=status 且 SLUG 已设置（已净化）时：

查找匹配 `*-{SLUG}` 模式的目录：
```bash
dir=$(ls -d .planning/quick/*-{SLUG}/ 2>/dev/null | head -1)
```

如果未找到目录，打印 `未找到 slug 为 {SLUG} 的 quick 任务` 并停止。

对于给定的 slug，读取 PLAN.md 和 SUMMARY.md（如果存在）。显示：
```
Quick Task: {slug}
─────────────────────────────────────
Plan 文件：.planning/quick/{dir}/PLAN.md
状态：{来自 SUMMARY.md frontmatter 的 status，或 "no summary yet"}
描述：{PLAN.md frontmatter 之后的第一行非空内容}
上次操作：{SUMMARY.md 的最后一行有意义内容，或 "none"}
─────────────────────────────────────
通过以下方式恢复：/gsd-quick resume {slug}
```

不启动 agent。打印后停止。

## RESUME 子命令

当 SUBCMD=resume 且 SLUG 已设置（已净化）时：

1. 查找匹配 `*-{SLUG}` 模式的目录：
   ```bash
   dir=$(ls -d .planning/quick/*-{SLUG}/ 2>/dev/null | head -1)
   ```
2. 如果未找到目录，打印 `未找到 slug 为 {SLUG} 的 quick 任务` 并停止。

3. 读取 PLAN.md 提取描述，读取 SUMMARY.md（如果存在）提取状态。

4. 启动前打印：
   ```
   [quick] 恢复中：.planning/quick/{dir}/
   [quick] Plan：{来自 PLAN.md 的描述}
   [quick] 状态：{来自 SUMMARY.md 的 status，或 "in-progress"}
   ```

5. 通过以下方式加载上下文：
   ```bash
   gsd-sdk query init.quick
   ```

6. 使用恢复上下文继续执行 quick 工作流，传递 slug 和计划目录，以便 executor 从上次中断处继续。

## RUN 子命令（默认）

当 SUBCMD=run 时：

端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/quick.md 中的 quick 工作流。
保留所有工作流关卡（验证、任务描述、规划、执行、状态更新、提交）。

</process>

<notes>
- Quick 任务驻留在 `.planning/quick/` 中 — 与阶段分离，不在 ROADMAP.md 中跟踪
- 每个 quick 任务获得一个 `YYYYMMDD-{slug}/` 目录，包含 PLAN.md 和最终的 SUMMARY.md
- STATE.md "Quick Tasks Completed" 表格在完成时更新
- 使用 `list` 审查累积的任务；使用 `resume` 继续进行中的工作
</notes>

<security_notes>
- 来自 $ARGUMENTS 的 slug 在用于文件路径之前进行净化：仅允许 [a-z0-9-]，最多 60 个字符，拒绝 ".." 和 "/"
- 来自 readdir/ls 的文件名在显示前进行净化：去除不可打印字符和 ANSI 序列
- 制品内容（计划描述、任务标题）仅作为纯文本渲染 — 永远不会执行，也不会在没有 DATA_START/DATA_END 边界的情况下传递给 agent prompt
- 状态字段通过 `gsd-sdk query frontmatter.get` 读取 — 永不 eval 或 shell 展开
</security_notes>
