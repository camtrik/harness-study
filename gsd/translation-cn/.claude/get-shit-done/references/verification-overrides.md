# 验证覆盖

当偏差已知且可接受时，有意接受 must-have 失败的机制。防止在按原规范永远不会通过的项目上出现验证循环。

<override_format>

## 覆盖格式

覆盖在 VERIFICATION.md 的 frontmatter 中的 `overrides:` 键下声明：

```yaml
---
phase: 03-authentication
verified: 2026-04-05T12:00:00Z
status: passed
score: 5/5
overrides_applied: 2
overrides:
  - must_have: "OAuth2 PKCE flow implemented"
    reason: "Using session-based auth instead — PKCE unnecessary for server-rendered app"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
  - must_have: "Rate limiting on login endpoint"
    reason: "Deferred to Phase 5 (infrastructure) — tracked in ROADMAP.md"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
---
```

### 必需字段

| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `must_have` | string | 被覆盖的 must-have truth、产物描述或 key link。不需要精确匹配——使用模糊匹配。 |
| `reason` | string | 为什么此偏差可接受。必须具体——不仅"不需要"。 |
| `accepted_by` | string | 谁接受了覆盖（用户名或角色）。必需。 |
| `accepted_at` | string | 覆盖被接受的 ISO 时间戳。必需。 |

</override_format>

## 何时使用

覆盖适用于阶段在执行期间有意偏离原始计划时——例如需求被缩减范围、选择了替代方法或依赖发生变化。

没有覆盖，verifier 将其报告为 FAIL，即使偏差是有意的。覆盖允许开发者将特定项目标记为 `PASSED (override)` 并附文档原因。

覆盖适用于以下情况：
- 需求在规划后变化但 ROADMAP.md 尚未更新
- 替代实现满足意图但不符合字面条文
- must-have 延迟到后续阶段并有显式跟踪
- 外部约束使原始 must-have 不可能或不必要

## 何时不使用

覆盖不适用于以下情况：
- 实现仅是不完整——修复它
- must-have 不清晰——澄清它
- 开发者想跳过验证——那会削弱流程
- 同一阶段多个 must-have 失败——如果需要覆盖超过 2-3 项，重新审视计划而非批量覆盖

<matching_rules>

## 匹配规则

覆盖匹配使用**模糊匹配**，而非精确字符串比较。这适应 must-have 在 ROADMAP.md、PLAN.md frontmatter 和覆盖条目中表达的微小措辞差异。

### 匹配算法

1. **规范化两个字符串：** 大小写不敏感比较——小写两个字符串，去除标点，压缩空白
2. **Token 重叠：** 分割为词，计算交集
3. **匹配阈值：** 在任一方向达到 80% token 重叠
4. **关键名词优先级：** 名词和技术术语权重高于常见词

### 歧义解决

如果覆盖匹配多个 must-have，将其应用于**最具体的匹配**（最高 token 重叠百分比）。如果仍然歧义，应用于第一个匹配并记录警告。

</matching_rules>

<verifier_behavior>

## 有覆盖时的 Verifier 行为

### 检查顺序

覆盖检查在**将 must-have 标记为 FAIL 之前**发生。流程是：

1. 根据代码库评估 must-have（验证流程的步骤 3-5）
2. 如果评估结果是 FAIL 或 UNCERTAIN：
   a. 在 VERIFICATION.md frontmatter 的 `overrides:` 数组中检查模糊匹配
   b. 如果找到覆盖：标记为 `PASSED (override)` 而非 FAIL
   c. 如果未找到覆盖：正常标记为 FAIL
3. 如果评估结果是 PASS：标记为 VERIFIED（覆盖无关）

### 输出格式

被覆盖的项目在所有验证表中以不同状态出现：

```markdown
| # | Truth | 状态 | 证据 |
|---|-------|--------|----------|
| 1 | 用户可以认证 | VERIFIED | OAuth 会话流工作 |
| 2 | OAuth2 PKCE flow | PASSED (override) | 覆盖：使用基于会话的认证——由 dave 于 2026-04-04 接受 |
| 3 | 聊天渲染消息 | FAILED | 组件返回占位符 |
```

### 对总体状态的影响

- `PASSED (override)` 项目计入通过分数，不计入失败分数
- 所有项目均为 VERIFIED 或 PASSED (override) 的阶段可以有状态 `passed`
- 覆盖不会抑制 `human_needed` 项目——那些仍然需要人工测试

</verifier_behavior>

<creating_overrides>

## 创建覆盖

### 交互式覆盖建议

当 verifier 将 must-have 标记为 FAIL 且失败看起来是有意的时，verifier 应建议创建覆盖：

```markdown
### F-002: OAuth2 PKCE 流

**状态：** FAILED
**证据：** 未找到 PKCE 实现。使用了基于会话的认证。

**这看起来是有意的。** 代码库使用基于会话的认证，以不同方式实现相同目标。要接受此偏差，向 VERIFICATION.md frontmatter 添加覆盖：
```

### 可通过验证工作流管理覆盖。

</creating_overrides>

<override_lifecycle>

## 覆盖生命周期

### 重新验证期间

当阶段重新验证时（例如差距关闭后）：
- 现有覆盖自动前移
- 如果底层代码现在满足 must-have，覆盖变得不必要——改为标记为 VERIFIED
- 覆盖永不会自动移除；它们作为文档保留

### 里程碑完成时

在 `/gsd-audit-milestone` 期间，覆盖呈现在审计报告中。这给团队在关闭里程碑前对所有接受偏差的可见性。

</override_lifecycle>

## 示例 VERIFICATION.md

```markdown
---
phase: 03-api-layer
verified: 2026-04-05T12:00:00Z
status: passed
score: 3/3
overrides_applied: 1
overrides:
  - must_have: "paginated API responses"
    reason: "Descoped — dataset under 100 items, pagination adds complexity without value"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
---

## 阶段 3: API 层 — 验证

| # | Truth | 状态 | 证据 |
|---|-------|--------|----------|
| 1 | REST 端点返回 JSON | VERIFIED | curl 测试确认 |
| 2 | 分页 API 响应 | PASSED (override) | 范围缩减——参见覆盖：数据集低于 100 项 |
| 3 | 认证中间件 | VERIFIED | JWT 验证工作 |
```
