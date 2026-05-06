<purpose>
对 UAT 和验证文件进行跨阶段审计。查找每个未完成项（待处理、已跳过、已阻塞、需要人工处理），可选择根据代码库进行验证以检测过时的文档，并生成优先排序的人工测试计划。
</purpose>

<process>

<step name="initialize">
运行 CLI 审计：

```bash
AUDIT=$(gsd-sdk query audit-uat --raw)
```

解析 JSON 获取 `results` 数组和 `summary` 对象。

如果 `summary.total_items` 为 0：显示"All Clear"并停止。
</step>

<step name="categorize">
将项目分组为现在可执行的 vs 需要先决条件的：

**现在可测试**（无外部依赖）：
- `pending` — 从未运行的测试
- `human_uat` — 人工验证项
- `skipped_unresolved` — 没有明确阻塞原因而跳过的

**需要先决条件：**
- `server_blocked` — 需要运行外部服务器
- `device_needed` — 需要物理设备（非模拟器）
- `build_needed` — 需要发布/预览构建
- `third_party` — 需要外部服务配置

对于"现在可测试"中的每个项目，检查底层功能在代码库中是否仍然存在（标记为 active/stale/needs_update）。
</step>

<step name="present">
呈现审计报告，包含现在可测试项、需要先决条件项和过时项的分组表格，以及推荐操作。
</step>

<step name="test_plan">
仅针对"现在可测试"且"active"的项目生成人工 UAT 测试计划，按可一起测试的内容分组。
</step>

</process>
