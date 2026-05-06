---
name: gsd:spec-phase
description: 通过歧义评分澄清阶段交付内容；在 discuss-phase 之前生成 SPEC.md
argument-hint: "<phase> [--auto] [--text]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
通过结构化苏格拉底式提问和量化歧义评分来澄清阶段需求。

**工作流中的定位：** `spec-phase → discuss-phase → plan-phase → execute-phase → verify`

**工作原理：**
1. 加载阶段上下文（PROJECT.md、REQUIREMENTS.md、ROADMAP.md、STATE.md）
2. 扫描代码库 — 在提问之前了解当前状态
3. 运行苏格拉底式访谈循环（最多 6 轮，轮换不同视角）
4. 每轮之后对 4 个加权维度进行歧义评分
5. 关卡：歧义 ≤ 0.20 且所有维度达到最低要求 → 写入 SPEC.md
6. 提交 SPEC.md — discuss-phase 在下次运行时自动拾取

**输出：** `{phase_dir}/{padded_phase}-SPEC.md` — 可证伪的需求，在 discuss-phase 处理"如何实现"之前锁定"做什么/为什么"
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spec-phase.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/spec.md
</execution_context>

<runtime_note>
**Copilot（VS Code）：** 在本工作流调用 `AskUserQuestion` 的地方使用 `vscode_askquestions`。它们是等效的。
</runtime_note>

<context>
阶段编号：$ARGUMENTS（必填）

**标志：**
- `--auto` — 跳过交互式问题；Claude 选择推荐的默认值并写入 SPEC.md
- `--text` — 使用纯文本编号列表而非 TUI 菜单（`/rc` 远程会话必需）

上下文文件在工作流中通过 `init phase-op` 解析。
</context>

<process>
端到端执行 @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/spec-phase.md 中的 spec-phase 工作流。

**强制要求：** 在执行任何操作之前读取工作流文件。工作流包含完整的逐步流程，包括苏格拉底式访谈循环、歧义评分关卡和 SPEC.md 生成。不要根据上述目标摘要即兴发挥。
</process>

<success_criteria>
- 提问开始前代码库已扫描当前状态
- 每次访谈循环后对全部 4 个歧义维度进行评分
- 关卡通过：歧义 ≤ 0.20 且所有维度最低要求已满足
- SPEC.md 已写入，包含可证伪的需求、明确的边界和验收标准
- SPEC.md 已原子提交
- 用户知晓现在可以运行 /gsd-discuss-phase，它将自动加载 SPEC.md
</success_criteria>
