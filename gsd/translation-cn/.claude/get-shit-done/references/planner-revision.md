# 修订模式——Planner 参考

当编排器提供带有检查器问题的 `<revision_context>` 时触发。不是从头开始——是对已有计划进行有针对性的更新。

**心态：** 外科医生，不是建筑师。针对特定问题进行最小改动。

### 步骤 1：加载已有计划

```bash
cat .planning/phases/$PHASE-*/$PHASE-*-PLAN.md
```

建立当前计划结构、已有任务、must_haves 的思维模型。

### 步骤 2：解析检查器问题

问题以结构化格式出现：

```yaml
issues:
  - plan: "16-01"
    dimension: "task_completeness"
    severity: "blocker"
    description: "Task 2 missing <verify> element"
    fix_hint: "Add verification command for build output"
```

按计划、维度、严重性分组。

### 步骤 3：修订策略

| 维度 | 策略 |
|-----------|----------|
| requirement_coverage | 为缺失的需求添加任务 |
| task_completeness | 向已有任务添加缺失元素 |
| dependency_correctness | 修复 depends_on，重新计算 wave |
| key_links_planned | 添加接入任务或更新 action |
| scope_sanity | 拆分为多个计划 |
| must_haves_derivation | 派生并添加 must_haves 到 frontmatter |

### 步骤 4：进行有针对性的更新

**要做的：** 编辑特定的被标记节，保留可工作的部分，如果依赖关系改变则更新 wave。

**不要做的：** 为小问题重写整个计划、添加不必要的任务、破坏已有可工作的计划。

### 步骤 5：验证更改

- [ ] 所有被标记的问题已处理
- [ ] 没有引入新问题
- [ ] Wave 编号仍然有效
- [ ] 依赖关系仍然正确
- [ ] 磁盘上的文件已更新

### 步骤 6：提交

```bash
gsd-sdk query commit "fix($PHASE): revise plans based on checker feedback" --files .planning/phases/$PHASE-*/$PHASE-*-PLAN.md
```

### 步骤 7：返回修订摘要

```markdown
## REVISION COMPLETE

**已处理的问题：** {N}/{M}

### 已进行的更改

| 计划 | 更改 | 处理的问题 |
|------|--------|-----------------|
| 16-01 | 向任务 2 添加了 <verify> | task_completeness |
| 16-02 | 添加了登出任务 | requirement_coverage (AUTH-02) |

### 已更新的文件

- .planning/phases/16-xxx/16-01-PLAN.md
- .planning/phases/16-xxx/16-02-PLAN.md

{如果有未处理的问题：}

### 未处理的问题

| 问题 | 原因 |
|-------|--------|
| {问题} | {为什么——需要用户输入、架构变更等} |
```
