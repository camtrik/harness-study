# --auto 模式 — 完全自主的讨论阶段

> **延迟加载。** 当 `$ARGUMENTS` 中存在 `--auto` 时，从 `workflows/discuss-phase.md` 读取此文件。讨论完成后，父文件的 `auto_advance` 步骤也会读取 `modes/chain.md` 来驱动自动推进到 plan-phase。

## 跨步骤效果

- **`check_existing`**：如果 CONTEXT.md 存在，自动选择"Update it" — 加载现有上下文并继续（匹配父步骤记录的 `--auto` 分支）。如果不存在上下文，不提示继续。对于中断的检查点，自动选择"Resume"。对于现有计划，自动选择"Continue and replan after"。记录每个决策以便用户审计。
- **`cross_reference_todos`**：自动折叠所有相关性评分 >= 0.4 的待办。记录选择。
- **`present_gray_areas`**：自动选择所有灰色区域。日志：`[--auto] Selected all gray areas: [区域名称列表]。`
- **`discuss_areas`**：对于每个讨论问题，**不使用 AskUserQuestion** 选择推荐选项（第一个选项，或明确标记为"recommended"的选项）。完全跳过交互提示。记录每个自动选择的选项以便用户在上下文文件中审查。
- 所有区域自动解决后，跳过"Explore more gray areas"提示，直接进入 `write_context`。
- 之后**自动推进**到 plan-phase。

## 关键 — 自动模式遍次上限

在 `--auto` 模式下，讨论步骤必须在**单次遍次**中完成。写入 CONTEXT.md 一次后，就完成了 — 立即进入 `write_context` 然后 auto_advance。不要重新读取自己的 CONTEXT.md 来寻找"差距"。这会造成自反馈循环。

从配置检查遍次上限：
```bash
MAX_PASSES=$(gsd-sdk query config-get workflow.max_discuss_passes 2>/dev/null || echo "3")
```

如果已经写入并提交了 CONTEXT.md，讨论步骤就完成了。继续前进。

## 组合规则

- `--auto --text` / `--auto --batch`：在自动模式下 text/batch 叠加是空操作（没有用户提示可渲染）。
- `--auto --analyze`：权衡表格仍可记录到审计追踪中；选择仍使用推荐选项。
- `--auto --power`：`--power` 胜出（power 模式为离线回答生成文件 — 与自主选择不兼容）。
