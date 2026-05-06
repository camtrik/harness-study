# 顾问模式 — 基于研究的比较表格

> **延迟加载和门控。** 父文件 `workflows/discuss-phase.md` 仅在 `ADVISOR_MODE` 为 true 时读取此文件（即当 `/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/USER-PROFILE.md` 存在时）。当没有 profile 时完全跳过读取 — 这是一个反转的 `--advisor` 标志设计（不在未使用时付出成本）。

## 激活

```bash
PROFILE_PATH="/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/USER-PROFILE.md"
if [ -f "$PROFILE_PATH" ]; then
  ADVISOR_MODE=true
else
  ADVISOR_MODE=false
fi
```

如果 `ADVISOR_MODE` 为 false，不要读取此文件 — 使用标准的 `default.md` 讨论流程继续。

## 校准层级

解析 `vendor_philosophy` 校准层级：
1. **优先级 1：** 读取 `config.json` > `preferences.vendor_philosophy`（项目级覆盖）
2. **优先级 2：** 读取 USER-PROFILE.md `Vendor Choices/Philosophy` 评分（全局）
3. **优先级 3：** 如果两者都没有值或值为 `UNSCORED`，则默认为 `"standard"`

映射到校准层级：
- `conservative` 或 `thorough-evaluator` → `full_maturity`
- `opinionated` → `minimal_decisive`
- `pragmatic-fast` 或任何其他值或空 → `standard`

解析顾问模型：
```bash
ADVISOR_MODEL=$(gsd-sdk query resolve-model gsd-advisor-researcher --raw)
```

## 非技术负责人检测

读取 USER-PROFILE.md 并检查产品负责人信号。如果存在以下任何一项，设置 `NON_TECHNICAL_OWNER = true`：
- `learning_style: guided`
- `frustration_triggers` 部分出现 `jargon` 一词
- `explanation_depth: practical-detailed`（无技术修饰符）
- `explanation_depth: high-level`

**打破平局/优先级（当信号冲突时）：**
1. 明确的 `technical_background: true` 覆盖所有推断的非技术信号 — 设置 `NON_TECHNICAL_OWNER = false`。
2. 否则，任何单个匹配信号足以设置 `NON_TECHNICAL_OWNER = true`（信号是 OR 聚合的，非加权）。
3. 矛盾的 `explanation_depth` 值：最新的条目胜出。

当 `NON_TECHNICAL_OWNER` 为 true 时，在呈现之前用产品结果语言重新表述灰色区域标签和描述。保留相同的底层决策 — 仅更改表述。

这适用于：
1. `present_gray_areas` 中的灰色区域标签和描述
2. 下面综合步骤中的顾问研究理由重写

## advisor_research 步骤

用户选择灰色区域后，并行启动研究 agent。

1. 显示简要状态：`Researching {N} areas...`

2. 为每个用户选择的灰色区域并行启动 `Task()`，传入灰色区域、阶段上下文、项目上下文和校准层级。

   所有 `Task()` 调用同时启动 — 不要等待一个完成再开始下一个。

   > **编排器规则 — CODEX 运行时**：调用所有 Task() 后，在 subagent 活动期间不要独立研究或分析任何灰色区域。等待所有 subagent 返回后再综合结果。

3. 所有 agent 返回后，**综合结果**再呈现：
   a. 解析 markdown 比较表格和理由段落
   b. 验证所有 5 列都存在（选项、优点、缺点、复杂度、推荐）— 填补缺失列
   c. 验证选项数量匹配校准层级
   d. 重写理由段落以融入 agent 没有的项目上下文和正在进行的讨论上下文
   e. 如果 agent 只返回 1 个选项，从表格格式转换为直接推荐
   f. **如果 `NON_TECHNICAL_OWNER` 为 true：** 对理由段落应用通俗语言重写

4. 存储综合表格以供 `discuss_areas` 使用（表格优先流程）。

## discuss_areas（顾问表格优先流程）

对于每个选定的区域：
1. 呈现综合的比较表格 + 理由段落
2. 使用 AskUserQuestion（或文本模式等效）从表格的选项列中提取选项
3. 记录用户的选择
4. 思维伙伴（条件性）：与默认模式相同
5. 决定是否需要后续问题
6. 所有区域处理后：询问是否创建上下文

## 范围蔓延处理（顾问模式）

如果用户提及阶段领域之外的内容，捕获为推迟的想法并重定向回当前讨论。
