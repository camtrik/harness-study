# --all 模式 — 自动选择所有灰色区域，交互式讨论

> **延迟加载。** 当 `$ARGUMENTS` 中存在 `--all` 时，从 `workflows/discuss-phase.md` 读取此文件。行为叠加在默认模式之上。

## 效果

- 在 `present_gray_areas` 中：自动选择所有灰色区域而无需询问用户（跳过 AskUserQuestion 区域选择步骤）。
- 每个区域的讨论**完全交互式**进行 — 用户驱动每个区域的每个问题（使用默认模式的 `discuss_areas` 流程）。
- 不会之后自动推进到 plan-phase — 如需自动推进使用 `--chain` 或 `--auto`。
- 日志：`[--all] Auto-selected all gray areas: [区域名称列表]。`

## 此模式存在的原因

这是"讨论一切"的快捷方式：跳过选择摩擦，保持对每个单独问题的完全交互控制。

## 组合规则

- `--all --auto`：`--auto` 在讨论阶段也胜出（Claude 选择推荐答案）；`--all` 的贡献仅是区域自动选择。
- `--all --chain`：区域自动选择，讨论交互式，然后自动推进到计划/执行（链语义）。
- `--all --batch` / `--all --text` / `--all --analyze`：分层叠加在讨论期间应用，如各自文件中所述。
