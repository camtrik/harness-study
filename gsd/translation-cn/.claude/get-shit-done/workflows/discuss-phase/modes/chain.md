# --chain 模式 — 交互式讨论，然后自动推进

> **延迟加载。** 当 `$ARGUMENTS` 中存在 `--chain` 时，或当父文件的 `auto_advance` 步骤需要在 `--auto` 下分派到 plan-phase 时，从 `workflows/discuss-phase.md` 读取此文件。

## 效果

- 讨论**完全交互式** — 问题、灰色区域选择和后续问题的行为与默认模式完全相同。
- 讨论完成后，**自动推进到 plan-phase → execute-phase**（与 `--auto` 相同的下游行为）。
- 这是中间地带：用户控制讨论决策，然后计划和执行自主运行。

## auto_advance 步骤（由父文件执行）

1. 从 `$ARGUMENTS` 解析 `--auto` 和 `--chain` 标志。注意：`--all` 不是自动推进触发器。

2. **将链标志与意图同步** — 如果用户手动调用（无 `--auto` 和 `--chain`），清除任何先前中断的 `--auto` 链的临时链标志。

3. 读取合并的自动模式状态。

4. 如果 `--auto` 或 `--chain` 标志存在且 `AUTO_MODE` 非 true：持久化链标志到配置。

5. 如果任何自动推进条件满足：显示横幅并启动 plan-phase 使用 Skill 工具。

6. **处理 plan-phase 返回：**
   - PHASE COMPLETE → 完整链成功
   - PLANNING COMPLETE → 计划完成，执行未完成
   - PLANNING INCONCLUSIVE / CHECKPOINT → 停止链，需要输入
   - GAPS FOUND → 停止链，发现差距

7. 如果没有任何自动推进条件：路由到 `confirm_creation` 步骤。
