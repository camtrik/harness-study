# 调试模板

`.planning/debug/[slug].md` 的模板——活动调试会话跟踪。

---

## 文件模板

```markdown
---
status: gathering | investigating | fixing | verifying | awaiting_human_verify | resolved
trigger: "[用户输入的原文]"
created: [ISO 时间戳]
updated: [ISO 时间戳]
---

## Current Focus
<!-- 每次更新时覆盖——始终反映当前状态 -->

hypothesis: [当前正在测试的假设]
test: [测试方法]
expecting: [结果为真/假时的含义]
next_action: [立即执行的下一步——要具体，不要写"继续调查"]
reasoning_checkpoint: null  <!-- 每次尝试修复前填充——见 structured_returns -->
tdd_checkpoint: null  <!-- tdd_mode 激活时在确认根因后填充 -->

## Symptoms
<!-- 在收集阶段编写，之后不可变 -->

expected: [应该发生什么]
actual: [实际发生了什么]
errors: [错误消息（如有）]
reproduction: [如何触发]
started: [何时开始出问题 / 一直有问题]

## Eliminated
<!-- 仅追加——防止 /clear 后重新调查 -->

- hypothesis: [错误的假设]
  evidence: [推翻它的证据]
  timestamp: [何时被排除]

## Evidence
<!-- 仅追加——调查中发现的事实 -->

- timestamp: [何时发现]
  checked: [检查了什么]
  found: [观察到了什么]
  implication: [这意味着什么]

## Resolution
<!-- 随理解深入而覆盖 -->

root_cause: [找到前为空]
fix: [应用前为空]
verification: [验证前为空]
files_changed: []
```

---

<section_rules>

**Frontmatter（status、trigger、timestamps）：**
- `status`：覆盖——反映当前阶段
- `trigger`：不可变——用户输入原文，永不更改
- `created`：不可变——设置一次
- `updated`：覆盖——每次更改时更新

**Current Focus：**
- 每次更新时完全覆盖
- 始终反映 Claude 当前正在做什么
- 如果 Claude 在 /clear 之后读取此内容，它能确切知道从哪恢复
- 字段：hypothesis、test、expecting、next_action、reasoning_checkpoint、tdd_checkpoint
- `next_action`：必须具体且可执行——不好的例子："继续调查"；好的例子："在 auth.js 第 47 行添加日志，观察 jwt.verify() 之前的 token 值"
- `reasoning_checkpoint`：每次 fix_and_verify 前覆盖——五字段结构化推理记录（假设、确认证据、证伪测试、修复理由、盲点）
- `tdd_checkpoint`：TDD 红/绿阶段覆盖——测试文件、名称、状态、失败输出

**Symptoms：**
- 在初始收集阶段编写
- 收集完成后不可变
- 作为我们试图修复的问题参考点
- 字段：expected、actual、errors、reproduction、started

**Eliminated：**
- 仅追加——永远不要删除条目
- 防止上下文重置后重新调查死胡同
- 每个条目：假设、推翻它的证据、时间戳
- 对于跨 /clear 边界的效率至关重要

**Evidence：**
- 仅追加——永远不要删除条目
- 调查中发现的事实
- 每个条目：时间戳、检查了什么、发现了什么、含义
- 为根因分析积累证据

**Resolution：**
- 随理解深入而覆盖
- 可能在尝试不同修复方案时多次更新
- 最终状态显示已确认的根因和已验证的修复
- 字段：root_cause、fix、verification、files_changed

</section_rules>

<lifecycle>

**创建：** 调用 /gsd-debug 时立即创建
- 使用用户输入的 trigger 创建文件
- 将 status 设为 "gathering"
- Current Focus：next_action = "收集症状"
- Symptoms：空，待填充

**症状收集期间：**
- 在用户回答问题时更新 Symptoms 段
- 每个问题更新 Current Focus
- 完成后：status → "investigating"

**调查期间：**
- 每个假设覆盖 Current Focus
- 每个发现追加到 Evidence
- 假设被推翻时追加到 Eliminated
- 更新 frontmatter 中的时间戳

**修复期间：**
- status → "fixing"
- 确认后更新 Resolution.root_cause
- 应用后更新 Resolution.fix
- 更新 Resolution.files_changed

**验证期间：**
- status → "verifying"
- 用结果更新 Resolution.verification
- 如果验证失败：status → "investigating"，重新尝试

**自我验证通过后：**
- status → "awaiting_human_verify"
- 在检查点中请求用户明确确认
- 暂时不要将文件移至 resolved

**解决时：**
- status → "resolved"
- 将文件移至 .planning/debug/resolved/（仅在用户确认修复后）

</lifecycle>

<resume_behavior>

当 Claude 在 /clear 后读取此文件：

1. 解析 frontmatter → 了解状态
2. 读取 Current Focus → 确切知道之前发生了什么
3. 读取 Eliminated → 知道什么不要重试
4. 读取 Evidence → 知道已经学到了什么
5. 从 next_action 继续

文件本身就是调试的大脑。Claude 应能从任何中断点完美恢复。

</resume_behavior>

<size_constraint>

保持调试文件聚焦：
- Evidence 条目：每个 1-2 行，只写事实
- Eliminated：简洁——假设 + 为什么失败
- 不要叙事性散文——仅结构化数据

如果证据积累得非常多（10+ 条目），考虑是否在原地打转。检查 Eliminated 确保没有重复尝试。

</size_constraint>
