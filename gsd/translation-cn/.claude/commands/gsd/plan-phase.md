---
name: gsd:plan-phase
description: 创建详细的阶段计划（PLAN.md），包含验证循环
argument-hint: "[phase] [--auto] [--research] [--skip-research] [--research-phase <N>] [--view] [--gaps] [--skip-verify] [--prd <file>] [--reviews] [--text] [--tdd] [--mvp]"
agent: gsd-planner
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
  - WebFetch
  - mcp__context7__*
---
<objective>
为路线图阶段创建可执行的阶段 prompt（PLAN.md 文件），包含集成的研究和验证。

**默认流程：** 研究（如需要）→ 计划 → 验证 → 完成

**仅研究模式（`--research-phase <N>`）：** 为阶段 `N` 启动 `gsd-phase-researcher`，写入 `RESEARCH.md`，然后在 planner 运行之前退出。适用于跨阶段研究、承诺规划方案之前进行文档审查，以及仅迭代研究而非重新启动 planner 的纠错不重规划循环。替换已删除的 `/gsd-research-phase` 命令（#3042）。

**仅研究模式修饰符：**
- **无标志** — 当 `RESEARCH.md` 已存在时，提示用户选择 `update / view / skip`。
- **`--research`** — 强制刷新：无条件重新启动 researcher。跳过现有的 RESEARCH.md 菜单。
- **`--view`** — 仅查看：将现有的 `RESEARCH.md` 打印到 stdout。不启动 researcher。对于纠错不重规划循环是最经济的模式。如果不存在 `RESEARCH.md`，则报错并提示去掉 `--view`。

**编排器角色：** 解析参数，验证阶段，研究领域（除非跳过），启动 gsd-planner，使用 gsd-plan-checker 验证，迭代直到通过或达到最大迭代次数，展示结果。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/plan-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。它们是等效的 — `vscode_askquestions` 是 VS Code Copilot 对同一交互式问题 API 的实现。不要因为 `AskUserQuestion` 看起来不可用就跳过提问步骤；改用 `vscode_askquestions`。
</runtime_note>

<context>
阶段编号：$ARGUMENTS（可选 — 省略时自动检测下一个未计划的阶段）

**标志：**
- `--research` — 即使 RESEARCH.md 存在也强制重新研究
- `--skip-research` — 跳过研究，直接进入规划
- `--gaps` — 差距关闭模式（读取 VERIFICATION.md，跳过研究）
- `--skip-verify` — 跳过验证循环
- `--prd <file>` — 使用 PRD/验收标准文件而非 discuss-phase。将需求自动解析为 CONTEXT.md。完全跳过 discuss-phase。
- `--reviews` — 重新规划，整合来自 REVIEWS.md 的跨 AI 审查反馈（由 `/gsd-review` 生成）
- `--text` — 使用纯文本编号列表而非 TUI 菜单（`/rc` 远程会话必需）
- `--mvp` — 垂直 MVP 模式。Planner 将任务组织为功能切片（UI→API→DB）而非水平层。在新项目的 Phase 1 上，还会生成 `SKELETON.md`（行走骨架）。可以通过 ROADMAP.md 中的 `**Mode:** mvp` 在阶段上持久化。

在步骤 2 的任何目录查找之前规范化阶段输入。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/plan-phase.md 中的 plan-phase 工作流。
保留所有工作流关卡（验证、研究、规划、验证循环、路由分发）。
</process>
