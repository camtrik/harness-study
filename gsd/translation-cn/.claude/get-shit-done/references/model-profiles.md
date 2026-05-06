# 模型配置文件

模型配置文件控制每个 GSD agent 使用哪个 Claude 模型。这允许在质量与 token 消耗之间取得平衡，或继承当前选择的会话模型。

## 配置文件定义

| Agent | `quality` | `balanced` | `budget` | `adaptive` | `inherit` |
|-------|-----------|------------|----------|------------|-----------|
| gsd-planner | opus | opus | sonnet | opus | inherit |
| gsd-roadmapper | opus | sonnet | sonnet | sonnet | inherit |
| gsd-executor | opus | sonnet | sonnet | sonnet | inherit |
| gsd-phase-researcher | opus | sonnet | haiku | sonnet | inherit |
| gsd-project-researcher | opus | sonnet | haiku | sonnet | inherit |
| gsd-research-synthesizer | sonnet | sonnet | haiku | haiku | inherit |
| gsd-debugger | opus | sonnet | sonnet | opus | inherit |
| gsd-codebase-mapper | sonnet | haiku | haiku | haiku | inherit |
| gsd-verifier | sonnet | sonnet | haiku | sonnet | inherit |
| gsd-plan-checker | sonnet | sonnet | haiku | haiku | inherit |
| gsd-integration-checker | sonnet | sonnet | haiku | haiku | inherit |
| gsd-nyquist-auditor | sonnet | sonnet | haiku | haiku | inherit |

## 按阶段类型的模型映射（#3023）

`.planning/config.json` 在 `models` 键下接受一个粗粒度的按**阶段类型**映射。当你希望在阶段级别进行调优（"规划和执行用 Opus，其余用 Sonnet"）而无需了解 agent 分类体系时使用。

```json
{
  "model_profile": "balanced",
  "models": {
    "planning": "opus",
    "discuss": "opus",
    "research": "sonnet",
    "execution": "opus",
    "verification": "sonnet",
    "completion": "sonnet"
  },
  "model_overrides": {
    "gsd-codebase-mapper": "haiku"
  }
}
```

### 阶段类型 → agent 映射

| 阶段类型 | Agent |
|---|---|
| `planning` | gsd-planner、gsd-roadmapper、gsd-pattern-mapper |
| `discuss` | （保留——目前无子 agent） |
| `research` | gsd-phase-researcher、gsd-project-researcher、gsd-research-synthesizer、gsd-codebase-mapper、gsd-ui-researcher |
| `execution` | gsd-executor、gsd-debugger、gsd-doc-writer |
| `verification` | gsd-verifier、gsd-plan-checker、gsd-integration-checker、gsd-nyquist-auditor、gsd-ui-checker、gsd-ui-auditor、gsd-doc-verifier |
| `completion` | （保留——目前无子 agent） |

### 解析优先级（从高到低）

1. **按 agent 的 `model_overrides[agent]`**——接受完整 ID；有针对性的例外
2. **阶段类型 `models[phase_type]`**——仅层级别名（`opus` / `sonnet` / `haiku` / `inherit`）
3. **配置文件表**——来自活跃 `model_profile` 的按 agent 列
4. **运行时默认值**——当没有其他适用时

### 为什么在配置文件之上有两层？

- **配置文件**是全局层级策略（所有人都运行 balanced）。
- **`models`** 是粗粒度的阶段级调优，不必学习 agent 名称。
- **`model_overrides`** 是每个 agent 的精确控制（例如为扇出强制 `gsd-codebase-mapper` 使用 `haiku`）。

三层组合：`models` 为阶段设置默认值，`model_overrides` 从中划出例外。

## 配置文件哲学

**quality**（最高质量）- 最大推理能力
- 所有决策 agent 使用 Opus
- 只读验证使用 Sonnet
- 何时使用：配额充足，关键架构工作

**balanced**（平衡，默认）- 智能分配
- 仅规划用 Opus（架构决策发生之处）
- 执行和研究用 Sonnet（遵循明确指令）
- 验证用 Sonnet（需要推理，不仅仅模式匹配）
- 何时使用：正常开发，质量与成本的良好平衡

**budget**（节省）- 最小 Opus 使用
- 任何写代码的用 Sonnet
- 研究和验证用 Haiku
- 何时使用：节省配额，大量工作，不太关键的阶段

**adaptive**（自适应）——基于角色的成本优化
- 规划和调试用 Opus（推理质量影响最大）
- 执行、研究和验证用 Sonnet（遵循明确指令）
- 映射、检查和审计用 Haiku（大量、结构化输出）
- 何时使用：在不牺牲计划质量的前提下优化成本，付费 API 层的个人开发

**inherit**（继承）- 跟随当前会话模型
- 所有 agent 解析为 `inherit`
- 最佳用于交互切换模型时（例如 OpenCode 或 Kilo `/model`）
- **使用非 Anthropic 供应商时必需**（OpenRouter、本地模型等）——否则 GSD 可能直接调用 Anthropic 模型，产生意外成本
- 何时使用：希望 GSD 跟随当前选择的运行时模型

## 使用非 Claude 运行时（Codex、OpenCode、Gemini CLI、Kilo）

当为非 Claude 运行时安装时，GSD 安装器在 `~/.gsd/defaults.json` 中设置 `resolve_model_ids: "omit"`。这会为所有 agent 返回空的模型参数，使每个 agent 使用运行时的默认模型。无需手动设置。

要为不同的 agent 分配不同的模型，添加 `model_overrides` 并使用你的运行时可识别的模型 ID：

```json
{
  "resolve_model_ids": "omit",
  "model_overrides": {
    "gsd-planner": "o3",
    "gsd-executor": "o4-mini",
    "gsd-debugger": "o3",
    "gsd-codebase-mapper": "o4-mini"
  }
}
```

相同的层级逻辑适用：规划和调试使用更强的模型，执行和映射使用更便宜的模型。

## 使用 Claude Code 与非 Anthropic 供应商（OpenRouter、本地）

如果你使用 Claude Code 与 OpenRouter、本地模型或任何非 Anthropic 供应商，设置 `inherit` 配置文件以防止 GSD 为子 agent 调用 Anthropic 模型：

```bash
# 通过 settings 命令
/gsd-settings
# → 为模型配置文件选择 "Inherit"

# 或手动在 .planning/config.json 中设置
{
  "model_profile": "inherit"
}
```

没有 `inherit`，GSD 的默认 `balanced` 配置文件会为每个 agent 类型生成特定的 Anthropic 模型（`opus`、`sonnet`、`haiku`），这可能通过你的非 Anthropic 供应商导致额外的 API 成本。

## 带失败层级升级的动态路由（#3024）

当 `.planning/config.json` 中 `dynamic_routing.enabled = true` 时，解析器根据 agent 的*默认层级*（light / standard / heavy）从层级映射表中选择模型，并在编排器检测到软失败时升级到下一层级。

```json
{
  "dynamic_routing": {
    "enabled": true,
    "tier_models": {
      "light":    "haiku",
      "standard": "sonnet",
      "heavy":    "opus"
    },
    "escalate_on_failure": true,
    "max_escalations": 1
  }
}
```

**Agent 默认层级**（`MODEL_PROFILES` 中的每个 agent 声明一个）：

| 层级 | Agent | 用例 |
|---|---|---|
| `light` | gsd-codebase-mapper、gsd-pattern-mapper、gsd-research-synthesizer、gsd-plan-checker、gsd-integration-checker、gsd-nyquist-auditor、gsd-ui-checker、gsd-ui-auditor、gsd-doc-verifier | 便宜/快速——纯映射器、扫描器、低风险审计 |
| `standard` | gsd-executor、gsd-phase-researcher、gsd-project-researcher、gsd-verifier、gsd-doc-writer、gsd-ui-researcher | 默认主力——研究、编写、主要验证 |
| `heavy` | gsd-planner、gsd-roadmapper、gsd-debugger | 深度推理——已在顶层，无法进一步升级 |

**升级流程**（编排器驱动）：

1. 编排器以 `attempt: 0` 生成 agent → 解析器返回 `tier_models[default_tier]`
2. 如果编排器将结果标记为软失败，以 `attempt: 1` 重新生成 → 解析器返回 `tier_models[next_tier_up]`
3. `max_escalations` 限制总重试次数（默认 1）。超过上限后解析器返回上限层级模型，以便编排器记录而不浪费更多预算。
4. 硬失败（异常）绕过升级，立即呈现。

**与其他层级源的优先级**（从高到低）：

1. `model_overrides[<agent>]`——完整 ID，始终优先
2. `dynamic_routing.tier_models[escalated_tier]`——当 `enabled: true` 时
3. `models[<phase_type>]`——粗粒度阶段级（#3023）
4. `model_profile`——全局层级策略

当 `dynamic_routing.enabled = false`（默认）时，行为与今天相同。

## 解析逻辑

编排器在生成之前解析模型。完整的优先层级（从高到低）是：

```text
1. 读取 .planning/config.json
2. 检查 model_overrides[<agent>]（接受完整 ID；有针对性的例外）
3. 如果 dynamic_routing.enabled，返回 tier_models[escalated_tier]
   （参见 §动态路由——升级步骤按尝试计数器层级上升）
4. 如果没有 dynamic_routing 匹配，检查 models[phase_type] 查找阶段类型层级
   （参见 §按阶段类型模型映射获取 agent → 阶段类型映射）
5. 如果没有阶段类型匹配，在配置文件表中查找 agent
6. 将模型参数传递给 Task 调用
```

相同的优先级适用于支持它的运行时（如 Codex）上的 `reasoning_effort` 解析，
因此 `model` 和 `reasoning_effort` 始终从相同的层级源派生——
`models[phase_type]` 或 `dynamic_routing` 覆盖会同时翻转两者。

## 按 Agent 覆盖

在不更改整个配置文件的情况下覆盖特定 agent：

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "opus",
    "gsd-planner": "haiku"
  }
}
```

覆盖优先于配置文件。有效值：`opus`、`sonnet`、`haiku`、`inherit`，或任何完全限定的模型 ID（例如 `"o3"`、`"openai/o3"`、`"google/gemini-2.5-pro"`）。

## 切换配置文件

运行时：`/gsd-set-profile <profile>`

按项目默认值：在 `.planning/config.json` 中设置：
```json
{
  "model_profile": "balanced"
}
```

## 设计理由

**为什么 gsd-planner 用 Opus？**
规划涉及架构决策、目标分解和任务设计。这是模型质量影响最高的地方。

**为什么 gsd-executor 用 Sonnet？**
executor 遵循明确的 PLAN.md 指令。计划已经包含推理；执行是实现。

**为什么 balanced 中的 verifier 用 Sonnet（而非 Haiku）？**
验证需要目标回溯推理——检查代码是否*交付*了阶段承诺的内容，而不仅仅是模式匹配。Sonnet 能很好处理；Haiku 可能遗漏微妙的差距。

**为什么 gsd-codebase-mapper 用 Haiku？**
只读探索和模式提取。不需要推理，只需要从文件内容中生成结构化输出。

**为什么返回 `inherit` 而不是直接传递 `opus`？**
Claude Code 的 `"opus"` 别名映射到特定模型版本。组织可能阻止旧版 opus 版本而允许新版。GSD 为 opus 级别的 agent 返回 `"inherit"`，使它们使用用户在会话中配置的任何 opus 版本。这避免了版本冲突和静默降级到 Sonnet。

**为什么有 `inherit` 配置文件？**
某些运行时（包括 OpenCode）允许用户在运行时切换模型（`/model`）。`inherit` 配置文件使所有 GSD 子 agent 与该实时选择保持一致。
