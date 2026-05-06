---
component: plan-checker
version: 1
last_calibrated: 2026-03-24
---

# Plan-Checker 少样本示例

## 正面示例

### 示例 1：标记缺少文件和函数名的模糊任务 action 为 BLOCKER

**输入：** 计划任务，action："Implement the authentication feature for the API routes."

**输出：**

> ```yaml
> issues:
>   - dimension: task_completeness
>     severity: BLOCKER
>     finding: "Task T1 action says 'implement the authentication feature' without naming target files, functions to create, or middleware to apply. Executor cannot determine what to build."
>     affected_field: "<action>"
>     suggested_fix: "Specify: create authMiddleware in src/middleware/auth.js, apply to routes in src/routes/api.js lines 12-45, verify with integration test"
> ```

**为什么好：** 检查器引用了具体维度（task_completeness），引用了有问题的文本，解释了为什么它是阻塞（executor 无法确定要构建什么），并给出了包含文件路径和函数名的具体修复。发现是可操作的——planner 确切知道要添加什么。

### 示例 2：两个计划在同一 wave 中修改同一文件的 BLOCKER

**输入：** Wave 1 中有两个计划——计划 01 修改 `get-shit-done/workflows/execute-phase.md`，计划 02 也修改 `get-shit-done/workflows/execute-phase.md`。未声明 `depends_on`。

**输出：**

> ```yaml
> issues:
>   - dimension: dependency_correctness
>     severity: BLOCKER
>     finding: "Plans 01 and 02 both modify get-shit-done/workflows/execute-phase.md in wave 1 with no depends_on relationship. Concurrent execution will cause merge conflicts or lost changes."
>     affected_field: "files_modified"
>     suggested_fix: "Either move Plan 02 to wave 2 with depends_on: ['01'] or consolidate the file changes into a single plan"
> ```

**为什么好：** 检查器识别了真正的结构问题——两个计划在没有依赖关系的情况下在同一 wave 中修改同一文件。它引用了 dependency_correctness，命名了两个计划、冲突的文件，并提供了两个替代修复。

## 负面示例

### 示例 1：缺少 verify 命令时所有维度均通过

**输入：** 包含 3 个任务的计划。任务 T2 有 `<verify></verify>`（空）。任务 T3 有 `<verify>echo "done"</verify>`（简单，无法区分通过和失败）。

**输出：**

> ## CHECK PASSED
> Plans: 1 | Tasks: 3 | Dimensions: 10 | Issues: 0

**为什么坏：** 检查器遗漏了两个违规。verification_derivation 要求 verify 命令可运行且测试实际输出——空的 verify 块是 BLOCKER。Verify 命令必须能区分通过和失败——`echo "done"` 无论实现状态如何都返回 0。两者都应被标记。

### 示例 2：正确大小计划的误报 INFO

**输入：** 包含 3 个任务、6 个唯一文件、1 个依赖的计划。全部在限制内。

**输出：**

> ```yaml
> issues:
>   - dimension: scope_sanity
>     severity: INFO
>     finding: "Plan has 3 tasks -- consider splitting into smaller plans for faster iteration"
>     affected_field: "task count"
>     suggested_fix: "Split tasks into separate plans"
> ```

**为什么坏：** 检查器标记了一个非问题。scope_sanity 允许每个计划 2-3 个任务——3 个任务在限制内。检查器应用了个人偏好（"越小越好"）而非文档记录的阈值。这浪费了 planner 在误报上的时间并侵蚀了对检查器判断的信任。正确的检查将不为该计划产生问题。
