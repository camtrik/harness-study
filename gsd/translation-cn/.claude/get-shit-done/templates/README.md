# GSD 规范产物注册表

此目录包含 GSD 工作流正式产出的每个产物的模板文件。下表是权威索引：**如果 `.planning/` 根目录文件未在此列出，`gsd-health` 将标记为 W019**（未识别的产物）。

Agent 在处理 `.planning/` 文件之前应查询此文件。如果文件名未出现在下面，则它不是规范的 GSD 产物。

---

## `.planning/` 根目录产物

这些文件直接位于 `.planning/` 下——不在阶段子目录内。

| 文件 | 模板 | 由哪个命令生成 | 用途 |
|------|----------|-------------|---------|
| `PROJECT.md` | `project.md` | `/gsd-new-project` | 项目身份、目标、需求摘要 |
| `ROADMAP.md` | `roadmap.md` | `/gsd-new-milestone`、`/gsd-new-project` | 带里程碑和进度跟踪的阶段计划 |
| `STATE.md` | `state.md` | `/gsd-new-project`、`/gsd-health --repair` | 当前会话状态、活跃阶段、最近活动 |
| `REQUIREMENTS.md` | `requirements.md` | `/gsd-new-milestone` | 带可追溯性的功能需求 |
| `MILESTONES.md` | `milestone.md` | `/gsd-complete-milestone` | 已完成的里程碑日志，含成就 |
| `BACKLOG.md` | *(内联)* | `/gsd-add-backlog` | 待处理的想法和已推迟的工作 |
| `LEARNINGS.md` | *(内联)* | `/gsd-extract-learnings`、`/gsd-execute-phase` | 阶段性回顾经验，供未来计划参考 |
| `THREADS.md` | *(内联)* | `/gsd-thread` | 持久化讨论线程 |
| `config.json` | `config.json` | `/gsd-new-project`、`/gsd-health --repair` | 项目特定的 GSD 配置 |
| `CLAUDE.md` | `claude-md.md` | `/gsd-profile` | 自动组装的 Claude Code 上下文文件 |

### 版本标记产物（模式：`vX.Y-*.md`）

| 模式 | 由哪个命令生成 | 用途 |
|---------|-------------|---------|
| `vX.Y-MILESTONE-AUDIT.md` | `/gsd-audit-milestone` | 归档前的里程碑审计报告 |

这些文件会被 `/gsd-complete-milestone` 归档到 `.planning/milestones/`。在完成归档后仍在 `.planning/` 根目录中找到这些文件，说明归档步骤被跳过了。

---

## 阶段子目录产物（`.planning/phases/NN-name/`）

这些文件位于阶段目录内。W019 不会检查它们（W019 仅检查 `.planning/` 根目录）。

| 文件模式 | 模板 | 由哪个命令生成 | 用途 |
|-------------|----------|-------------|---------|
| `NN-MM-PLAN.md` | `phase-prompt.md` | `/gsd-plan-phase` | 可执行的实现计划 |
| `NN-MM-SUMMARY.md` | `summary.md` | `/gsd-execute-phase` | 执行后的摘要，含经验教训 |
| `NN-CONTEXT.md` | `context.md` | `/gsd-discuss-phase` | 阶段的范围化讨论决策 |
| `NN-RESEARCH.md` | `research.md` | `/gsd-plan-phase`、`/gsd-plan-phase --research-phase <N>` | 阶段的技术研究 |
| `NN-VALIDATION.md` | `VALIDATION.md` | `/gsd-plan-phase`（Nyquist） | 验证架构（Nyquist 方法） |
| `NN-UAT.md` | `UAT.md` | `/gsd-validate-phase` | 用户验收测试结果 |
| `NN-PATTERNS.md` | *(内联)* | `/gsd-plan-phase`（pattern mapper） | 阶段的类比文件映射 |
| `NN-UI-SPEC.md` | `UI-SPEC.md` | `/gsd-ui-phase` | UI 设计合约 |
| `NN-SECURITY.md` | `SECURITY.md` | `/gsd-secure-phase` | 安全威胁模型 |
| `NN-AI-SPEC.md` | `AI-SPEC.md` | `/gsd-ai-integration-phase` | 含评估策略的 AI 集成规范 |
| `NN-DEBUG.md` | `DEBUG.md` | `/gsd-debug` | 调试会话日志 |
| `NN-REVIEWS.md` | *(内联)* | `/gsd-review` | 跨 AI 审查反馈 |

---

## 里程碑归档（`.planning/milestones/`）

由 `/gsd-complete-milestone` 归档的文件。这些文件永远不会被 W019 检查。

| 文件模式 | 来源 |
|-------------|--------|
| `vX.Y-ROADMAP.md` | 里程碑关闭时 ROADMAP.md 的快照 |
| `vX.Y-REQUIREMENTS.md` | 里程碑关闭时 REQUIREMENTS.md 的快照 |
| `vX.Y-MILESTONE-AUDIT.md` | 从 `.planning/` 根目录移入 |
| `vX.Y-phases/` | 已归档的阶段目录（如果使用了 `--archive-phases`） |

---

## 添加新的规范产物

当新工作流生成 `.planning/` 根目录文件时：

1. 将文件名添加到 `get-shit-done/bin/lib/artifacts.cjs` 中的 `CANONICAL_EXACT`
2. 在上方的 **`.planning/` 根目录产物** 表格中添加一行
3. 如果有对应的模板，将其添加到 `get-shit-done/templates/` 中
