---
component: verifier
version: 2
last_calibrated: 2026-03-25
calibration_source: "Adapted from thinking-partner verification corpus"
---

# Verifier 少样本示例

## 正面示例

### 示例 1：通过多层产物验证发现真实差距

**输入：** Must-have 产物：`get-shit-done/workflows/context-bridge.md: >50 lines`

**输出：**

> **L1（存在性）：** EXISTS——找到 `get-shit-done/workflows/context-bridge.md`，187 行。
> **L2（实质性）：** SUBSTANTIVE——无 TODO/FIXME 标记，4 个已定义的部分（`updateContextTier`、`getContextBudget`、`injectContextWarning`、`resetTierState`），非平凡逻辑，含配置读取和阈值计算。
> **L3（已接入）：** WIRED——由 `get-shit-done/workflows/execute-phase.md` 在步骤 3 引用。由 `agents/gsd-executor.md` 通过 @ 引用包含调用。参数已验证：接收与预期参数匹配的规划目录路径。
> **状态：PASSED**

**为什么好：** Verifier 在每一层独立检查，并在每层提供具体证据——文件路径、行数、导出符号、导入位置和参数验证。它没有停留在"文件存在"而是追踪接入到实际使用及正确参数。

### 示例 2：通过 grep 证据正确分类 FAILED

**输入：** Must-have truth：`validate-commit.js 拒绝缺少 scope 括号的提交`

**输出：**

> **方法：** 运行 `grep -n "scope" get-shit-done/bin/validate-commit.js` 定位 scope 验证逻辑。
> **证据：** 第 34-41 行显示正则 `/^(feat|fix|refactor|test|docs|chore|wip)\(/` 要求在类型后跟开括号。然而，没有测试缺失闭括号的情况——正则接受 `feat(: description`（缺少 scope 内容和闭括号）。
> **运行：** `echo "feat(: test" | node get-shit-done/bin/validate-commit.js`——exit code 0（接受）。
> **状态：FAILED**——带空 scope 的提交消息 `feat(: desc)` 被接受。"拒绝缺少 scope 括号的提交"这一 truth 在空 scope 边界情况下不成立。

**为什么好：** Verifier 不相信代码的意图——它用边界情况输入运行了实际测试并报告了具体失败。证据包括正则、测试命令、exit code 和为什么 truth 不成立。

### 示例 3：发现缺失接入（校准派生的差距模式）

**输入：** Must-have key_link：`check-subagent-output.js 导入并调用 feedback-loop.js 的 extractFeedback()`

**输出：**

> **方法：** 运行 `grep -n "feedback-loop\|extractFeedback" get-shit-done/bin/check-subagent-output.js`。
> **证据：** 零匹配。文件 `feedback-loop.js` 存在（172 行，实质性），`tests/feedback-loop.test.js` 通过（229 行）。但 `check-subagent-output.js` 从未导入或调用它。
> **交叉检查：** `grep -rn "require.*feedback-loop" get-shit-done/bin/` 在所有脚本中返回零结果。
> **状态：FAILED**——产物存在且被测试，但完全孤立。key_link"导入并调用"不成立。这是一个**缺失接入**的差距。

**为什么好：** Verifier 发现了一个常见的校准模式：存在、被测试但从未实际接入系统的代码。L1 和 L2 通过，但 L3（接入）失败。此模式占校准语料库中发现的所有差距的 37%。

### 示例 4：发现缺失配置模式条目（校准派生的差距模式）

**输入：** Must-have truth：`Config toggles features.architecture_graph and features.architecture_guard exist with defaults`

**输出：** 发现默认值在代码中存在但模式验证不知道它们——**缺失接入**的配置层之间的差距。

**为什么好：** Verifier 检查了多个配置层（模式 vs 默认值 vs 代码）而不仅仅一层。配置接入要求模式 + 默认值 + 代码一致——仅检查一层会遗漏层不匹配 bug。

### 示例 5：区分规划差距和执行失败

**输入：** Must-have truth：`所有 MSYS 正则模式合并到共享模块中`

**输出：** 发现 2 个匹配项位于计划范围外的文件中——在计划创建后提取代码。executor 正确完成了所有计划任务。差距是**规划排序问题**，而非执行失败。

**为什么好：** Verifier 区分了什么是计划的和什么现在存在。计划创建后改变的代码是规划差距，而非 executor 失败。这防止了对正确执行的工作的错误 FAIL 判断。

---

## 校准派生的差距模式

基于验证语料库分析（80% 通过率，8 个差距）：

| 模式 | 计数 | 占差距百分比 | 检查什么 |
|---------|-------|-----------|---------------|
| 缺失接入 | 3 | 37% | 文件存在 + 被测试但从未被消费者导入/调用 |
| 缺失测试 | 2 | 25% | 功能工作但测试文件从 CI 排除或从未运行 |
| 其他 | 3 | 38% | 配置层不匹配、规划排序问题、模式差距 |

## 负面示例

### 示例 1：无按标准证据的全盘通过

**输入：** Must-haves：来自计划 frontmatter 的 5 个 truth、3 个产物、2 个 key_link。

**输出：**

> All must-haves have been verified. The implementation matches the plan requirements. Files exist and are properly connected. Status: PASSED.
> must_haves_checked: 10, must_haves_passed: 10

**为什么坏：** 没有列出或检查任何一个单独的 must-have。没有文件路径、没有行号、没有 grep 输出、没有逐层分解。Verifier 将 must-have 重复为通过而不显示任何证据。这会遗漏桩、孤立导出和破损接入。

### 示例 2：无实质或接入验证的文件存在性检查

**输入：** Must-have 产物：`get-shit-done/bin/lib/audit-checks/index.cjs: >100 lines`

**输出：**

> **L1（存在性）：** EXISTS——文件找到于 `get-shit-done/bin/lib/audit-checks/index.cjs`，142 行。
> **状态：PASSED**

**为什么坏：** Verifier 停留在第 1 层。文件有 142 行但可能包含 `// TODO: implement all checks` 并带有返回空对象的桩函数。第 2 层（实质性）和第 3 层（已接入）完全被跳过。存在但从未导入或仅包含占位符代码的文件不应通过。
