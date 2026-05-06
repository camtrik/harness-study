# UAT 模板

`.planning/phases/XX-name/{phase_num}-UAT.md` 的模板——持久化 UAT 会话跟踪。

---

## 文件模板

```markdown
---
status: testing | partial | complete | diagnosed
phase: XX-name
source: [已测试的 SUMMARY.md 文件列表]
started: [ISO 时间戳]
updated: [ISO 时间戳]
---

## Current Test
<!-- 每个测试覆盖——显示当前进度 -->

number: [N]
name: [测试名称]
expected: |
  [用户应观察到什么]
awaiting: 用户回复

## Tests

### 1. [测试名称]
expected: [可观察行为——用户应看到什么]
result: [pending]

### 2. [测试名称]
expected: [可观察行为]
result: pass

### 3. [测试名称]
expected: [可观察行为]
result: issue
reported: "[用户回复原文]"
severity: major

### 4. [测试名称]
expected: [可观察行为]
result: skipped
reason: [为什么跳过]

### 5. [测试名称]
expected: [可观察行为]
result: blocked
blocked_by: server | physical-device | release-build | third-party | prior-phase
reason: [为什么阻塞]

...

## Summary

total: [N]
passed: [N]
issues: [N]
pending: [N]
skipped: [N]
blocked: [N]

## Gaps

<!-- YAML 格式供 plan-phase --gaps 消费 -->
- truth: "[测试中预期的行为]"
  status: failed
  reason: "用户报告：[原文回复]"
  severity: blocker | major | minor | cosmetic
  test: [N]
  root_cause: ""     # 由诊断填充
  artifacts: []      # 由诊断填充
  missing: []        # 由诊断填充
  debug_session: ""  # 由诊断填充
```

---

<section_rules>

**Frontmatter：**
- `status`：覆盖——"testing"、"partial" 或 "complete"
- `phase`：不可变——创建时设置
- `source`：不可变——正在测试的 SUMMARY 文件
- `started`：不可变——创建时设置
- `updated`：覆盖——每次更改时更新

**Current Test：**
- 每次测试转换时完全覆盖
- 显示哪个测试活跃及等待什么
- 完成后："[testing complete]"

**Tests：**
- 每个测试：用户回复时覆盖 result 字段
- `result` 值：[pending]、pass、issue、skipped、blocked
- 如果是 issue：添加 `reported`（原文）和 `severity`（推断）
- 如果是 skipped：添加 `reason`（如有提供）
- 如果是 blocked：添加 `blocked_by`（标签）和 `reason`（如有提供）

**Summary：**
- 每次回复后覆盖计数
- 跟踪：total、passed、issues、pending、skipped、blocked

**Gaps：**
- 发现 issue 时仅追加（YAML 格式）
- 诊断后：填充 `root_cause`、`artifacts`、`missing`、`debug_session`
- 此段直接输入 /gsd-plan-phase --gaps

</section_rules>

<diagnosis_lifecycle>

**测试完成后（status: complete），如果存在缺口：**

1. 用户运行诊断（来自 verify-work 的提示或手动）
2. diagnose-issues 工作流生成并行 debug agent
3. 每个 agent 调查一个缺口，返回根因
4. UAT.md Gaps 段更新诊断信息：
   - 每个缺口填充 `root_cause`、`artifacts`、`missing`、`debug_session`
5. status → "diagnosed"
6. 可进行 /gsd-plan-phase --gaps 修复

**诊断后：**
```yaml
## Gaps

- truth: "评论在提交后立即出现"
  status: failed
  reason: "用户报告：功能可用但刷新页面后才显示"
  severity: major
  test: 2
  root_cause: "CommentList.tsx 中的 useEffect 缺少 commentCount 依赖"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect 缺少依赖"
  missing:
    - "将 commentCount 添加到 useEffect 依赖数组"
  debug_session: ".planning/debug/comment-not-refreshing.md"
```

</diagnosis_lifecycle>

<lifecycle>

**创建：** 当 /gsd-verify-work 启动新会话时
- 从 SUMMARY.md 文件中提取测试
- 将 status 设为 "testing"
- Current Test 指向测试 1
- 所有测试的 result 为：[pending]

**测试期间：**
- 从 Current Test 段呈现测试
- 用户回复通过确认或问题描述
- 更新测试 result（pass/issue/skipped）
- 更新 Summary 计数
- 如果是 issue：追加到 Gaps 段（YAML 格式），推断严重性
- 将 Current Test 移至下一个待处理测试

**完成时：**
- status → "complete"
- Current Test → "[testing complete]"
- 提交文件
- 呈现摘要及下一步

**部分完成：**
- status → "partial"（如果有 pending、blocked 或未解决的 skipped 测试）
- Current Test → "[testing paused — {N} items outstanding]"
- 提交文件
- 呈现摘要，高亮显示未完成项

**恢复部分会话：**
- `/gsd-verify-work {phase}` 从第一个 pending/blocked 测试继续
- 当所有项解决后，status 变为 "complete"

**在 /clear 后恢复：**
1. 读取 frontmatter → 了解阶段和状态
2. 读取 Current Test → 知道我们到哪了
3. 找到第一个 [pending] result → 从那里继续
4. Summary 显示当前进度

</lifecycle>

<severity_guide>

严重性从用户的自然语言中推断，绝不主动询问。

| 用户描述 | 推断 |
|----------------|-------|
| 崩溃、错误、异常、完全失败、不可用 | blocker |
| 不工作、什么都没发生、行为错误、缺失 | major |
| 能用但是...、慢、奇怪、微小、小问题 | minor |
| 颜色、字体、间距、对齐、视觉、外观问题 | cosmetic |

默认：**major**（安全默认值，用户可在不对时澄清）

</severity_guide>

<good_example>
```markdown
---
status: diagnosed
phase: 04-comments
source: 04-01-SUMMARY.md, 04-02-SUMMARY.md
started: 2025-01-15T10:30:00Z
updated: 2025-01-15T10:45:00Z
---

## Current Test

[testing complete]

## Tests

### 1. 查看帖子评论
expected: 评论区域展开，显示计数和评论列表
result: pass

### 2. 创建顶级评论
expected: 通过富文本编辑器提交评论，显示在列表中，含作者信息
result: issue
reported: "能用但刷新页面后才显示"
severity: major

### 3. 回复评论
expected: 点击回复，内联编辑器出现，提交显示嵌套回复
result: pass

### 4. 视觉嵌套
expected: 3+ 级线程显示缩进、左边框，合理深度处封顶
result: pass

### 5. 删除自己的评论
expected: 点击自己评论上的删除，被移除或显示 [deleted]（如有回复）
result: pass

### 6. 评论计数
expected: 帖子显示准确计数，添加评论时递增
result: pass

## Summary

total: 6
passed: 5
issues: 1
pending: 0
skipped: 0
blocked: 0

## Gaps

- truth: "评论在列表提交后立即出现"
  status: failed
  reason: "用户报告：能用但刷新页面后才显示"
  severity: major
  test: 2
  root_cause: "CommentList.tsx 中的 useEffect 缺少 commentCount 依赖"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect 缺少依赖"
  missing:
    - "将 commentCount 添加到 useEffect 依赖数组"
  debug_session: ".planning/debug/comment-not-refreshing.md"
```
</good_example>
