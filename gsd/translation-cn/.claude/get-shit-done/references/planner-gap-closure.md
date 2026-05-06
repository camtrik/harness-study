# 差距关闭模式——Planner 参考

由 `--gaps` 标志触发。创建处理验证或 UAT 失败的计划。

**重要：跳过延迟项。** 读取 VERIFICATION.md 时，仅 `gaps:` 节包含需要关闭计划的可操作项。`deferred:` 节（如果存在）列出的是已在后续里程碑阶段显式处理的项目——这些不是差距，在差距关闭规划期间必须被忽略。为延迟项创建计划会浪费在已为未来阶段安排的工作上的努力。

**1. 查找差距来源：**

使用 init 上下文（来自 load_project_state），它提供 `phase_dir`：

```bash
# 检查 VERIFICATION.md（代码验证差距）
ls "$phase_dir"/*-VERIFICATION.md 2>/dev/null

# 检查具有 diagnosed 状态的 UAT.md（用户测试差距）
grep -l "status: diagnosed" "$phase_dir"/*-UAT.md 2>/dev/null
```

**2. 解析差距：** 每个差距有：truth（失败的行为）、reason、artifacts（有问题的文件）、missing（需要添加/修复的内容）。

**3. 加载已有的 SUMMARY** 以理解已构建的内容。

**4. 查找下一个计划编号：** 如果计划 01-03 存在，下一个是 04。

**5. 将差距分组为计划**：按相同产物、相同关注点、依赖顺序（如果产物是桩则无法接入 → 先修复桩）。

**6. 创建差距关闭任务：**

```xml
<task name="{修复描述}" type="auto">
  <files>{artifact.path}</files>
  <action>
    {对于 gap.missing 中的每一项：}
    - {缺失项}

    引用已有代码：{来自 SUMMARY}
    差距原因：{gap.reason}
  </action>
  <verify>{如何确认差距已关闭}</verify>
  <done>{现在可达成可观察的 truth}</done>
</task>
```

**7. 使用标准依赖分析分配 wave**（与 `assign_waves` 步骤相同）：
- 无依赖的计划 → wave 1
- 依赖其他差距关闭计划的计划 → max(依赖的 wave) + 1
- 同时考虑对阶段中已有（非差距）计划的依赖

**8. 写入 PLAN.md 文件：**

```yaml
---
phase: XX-name
plan: NN              # 在已有之后顺序递增
type: execute
wave: N               # 根据 depends_on 计算（参见 assign_waves）
depends_on: [...]     # 此计划依赖的其他计划（差距或已有）
files_modified: [...]
autonomous: true
gap_closure: true     # 用于跟踪的标志
---
```
