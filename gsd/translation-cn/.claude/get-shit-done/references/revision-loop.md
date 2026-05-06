# 修订循环模式

迭代 agent 修订与反馈的标准模式。当检查器/验证器发现问题且产出 agent 需要修订其输出时使用。

---

## 模式：检查-修订-升级（最多 3 次迭代）

此模式适用于：
1. agent 产生输出（计划、导入、差距关闭计划）
2. 检查器/验证器评估该输出
3. 发现需要修订的问题

### 流程

```
prev_issue_count = Infinity
iteration = 0

循环：
  1. 对当前输出运行检查器/验证器
  2. 读取检查器结果
  3. 如果通过或仅有 INFO 级别问题：
     -> 接受输出，退出循环
  4. 如果发现 BLOCKER 或 WARNING 问题：
     a. iteration += 1
     b. 如果 iteration > 3：
        -> 升级给用户（见下方"3 次迭代后"）
     c. 从检查器输出中解析问题计数
     d. 如果 issue_count >= prev_issue_count：
        -> 升级给用户："修订循环停滞（问题计数未减少）"
     e. prev_issue_count = issue_count
     f. 重新生成产出 agent，附带检查器反馈
     g. 修订完成后，回到循环
```

### 问题计数跟踪

跟踪检查器在每次迭代中返回的 BLOCKER + WARNING 问题数量。如果连续迭代之间计数不减少，说明产出 agent 陷入困境，进一步迭代将没有帮助。提前中断并升级给用户。

在每次修订生成前显示迭代进度：
`修订迭代 {N}/3——{blocker_count} 个阻塞，{warning_count} 个警告`

### 重新生成 Prompt 结构

当为修订重新生成产出 agent 时，传递检查器的 YAML 格式问题。

```
<checker_issues>
以下是 YAML 格式的问题。每个有：dimension、severity、finding、
affected_field、suggested_fix。处理所有 BLOCKER 问题。尽可能处理 WARNING 问题。

{来自检查器输出的 YAML 问题块——原样传递}
</checker_issues>

<revision_instructions>
处理上述所有 BLOCKER 和 WARNING 问题。
- 对每个 BLOCKER：进行所需的更改
- 对每个 WARNING：处理或解释为什么可接受
- 修复时不要引入新问题
- 保留所有未被检查器标记的内容
这是第 {N}/3 次修订迭代。上一次迭代有 {prev_count} 个问题。
你必须减少计数，否则循环将终止。
</revision_instructions>
```

### 3 次迭代后

如果 3 个修订周期后问题仍然存在：

1. 将剩余问题呈现给用户
2. 使用门提示（来自 `references/gate-prompts.md` 的 yes-no 模式）
3. 如果"继续"：接受当前输出并继续
4. 如果"调整方法"或"其他"：与用户讨论，然后以更新的上下文重新进入产出步骤

### 工作流特定变体

| 工作流 | 产出 Agent | 检查器 Agent | 备注 |
|----------|---------------|---------------|-------|
| plan-phase | gsd-planner | gsd-plan-checker | 通过 planner-revision.md 提供修订提示 |
| execute-phase | gsd-executor | gsd-verifier | 执行后验证 |
| discuss-phase | 编排器 | gsd-plan-checker | 编排器行内修订 |

---

## 重要说明

- **INFO 级别问题始终可接受**——它们不触发修订
- **每次迭代生成新的 agent**——不要尝试在同一上下文中继续
- **检查器反馈必须内联**——修订 agent 需要看到确切失败的内容
- **不要静默吞没问题**——始终在退出循环后将最终状态呈现给用户
