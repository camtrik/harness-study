<overview>
TDD 关乎设计质量，而非覆盖率指标。红-绿-重构循环迫使你在实现之前思考行为，产生更清晰的接口和更可测试的代码。

**原则：** 如果你能在写 `fn` 之前将行为描述为 `expect(fn(input)).toBe(output)`，TDD 会改善结果。

**关键洞察：** TDD 工作本质上比标准任务更重——它需要 2-3 个执行循环（RED → GREEN → REFACTOR），每个都涉及文件读取、测试运行和潜在的调试。TDD 功能获得专用计划以确保整个循环中有完整的上下文。
</overview>

<when_to_use_tdd>
## TDD 何时能提高质量

**TDD 候选人（创建 TDD 计划）：**
- 具有定义输入/输出的业务逻辑
- 具有请求/响应合约的 API 端点
- 数据转换、解析、格式化
- 验证规则和约束
- 具有可测试行为的算法
- 状态机和工作流
- 具有明确规格的工具函数

**跳过 TDD（使用标准计划的 `type="auto"` 任务）：**
- UI 布局、样式、视觉组件
- 配置更改
- 连接现有组件的胶水代码
- 一次性脚本和迁移
- 无业务逻辑的简单 CRUD
- 探索性原型

**启发式：** 你能在写 `fn` 之前写出 `expect(fn(input)).toBe(output)` 吗？
→ 是：创建 TDD 计划
→ 否：使用标准计划，需要时之后添加测试
</when_to_use_tdd>

<tdd_plan_structure>
## TDD 计划结构

每个 TDD 计划通过完整的 RED-GREEN-REFACTOR 循环实现**一个功能**。

```markdown
---
phase: XX-name
plan: NN
type: tdd
---

<objective>
[什么功能以及为什么]
目的：[此功能 TDD 的设计收益]
输出：[可工作、已测试的功能]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@relevant/source/files.ts
</context>

<feature>
  <name>[功能名称]</name>
  <files>[源文件、测试文件]</files>
  <behavior>
    [以可测试术语描述的预期行为]
    用例：输入 → 预期输出
  </behavior>
  <implementation>[测试通过后如何实现]</implementation>
</feature>

<verification>
[证明功能工作的测试命令]
</verification>

<success_criteria>
- 失败的测试已编写并提交
- 实现通过测试
- 重构完成（如果需要）
- 所有 2-3 个 commit 都存在
</success_criteria>

<output>
完成后，创建 SUMMARY.md 包含：
- RED：编写了什么测试，为什么它失败
- GREEN：什么实现使其通过
- REFACTOR：完成了什么清理（如果有）
- 提交：产出的 commit 列表
</output>
```

**每个 TDD 计划一个功能。** 如果功能太简单可以批量处理，那也太简单不值得 TDD——使用标准计划并在之后添加测试。
</tdd_plan_structure>

<execution_flow>
## 红-绿-重构循环

**RED——编写失败测试：**
1. 按照项目惯例创建测试文件
2. 编写描述预期行为的测试（来自 `<behavior>` 元素）
3. 运行测试——必须失败
4. 如果测试通过：功能已存在或测试错误。调查。
5. 提交：`test({phase}-{plan}): add failing test for [feature]`

**GREEN——实现以达到通过：**
1. 编写最少代码使测试通过
2. 无聪明、无优化——只是让它工作
3. 运行测试——必须通过
4. 提交：`feat({phase}-{plan}): implement [feature]`

**REFACTOR（如果需要）：**
1. 如果有明显改进，清理实现
2. 运行测试——必须仍然通过
3. 仅在做了更改时提交：`refactor({phase}-{plan}): clean up [feature]`

**结果：** 每个 TDD 计划产出 2-3 个原子 commit。
</execution_flow>

<test_quality>
## 好测试与坏测试

**测试行为，而非实现：**
- 好："返回格式化日期字符串"
- 坏："使用正确参数调用 formatDate 辅助函数"
- 测试应能在重构后依然存活

**每个测试一个概念：**
- 好：为有效输入、空输入、格式错误输入分别编写测试
- 坏：单个测试检查所有边界情况，使用多个断言

**描述性命名：**
- 好："should reject empty email"、"returns null for invalid ID"
- 坏："test1"、"handles error"、"works correctly"

**无实现细节：**
- 好：测试公共 API、可观察行为
- 坏：Mock 内部、测试私有方法、断言内部状态
</test_quality>

<framework_setup>
## 测试框架设置（如果不存在）

当执行 TDD 计划但没有配置测试框架时，作为 RED 阶段的一部分设置：

**1. 检测项目类型：**
```bash
# JavaScript/TypeScript
if [ -f package.json ]; then echo "node"; fi

# Python
if [ -f requirements.txt ] || [ -f pyproject.toml ]; then echo "python"; fi

# Go
if [ -f go.mod ]; then echo "go"; fi

# Rust
if [ -f Cargo.toml ]; then echo "rust"; fi
```

**2. 安装最小框架：**
| 项目 | 框架 | 安装 |
|---------|-----------|---------|
| Node.js | Jest | `npm install -D jest @types/jest ts-jest` |
| Node.js（Vite） | Vitest | `npm install -D vitest` |
| Python | pytest | `pip install pytest` |
| Go | testing | 内置 |
| Rust | cargo test | 内置 |

**3. 创建配置（如果需要）：**
- Jest：`jest.config.js` 搭配 ts-jest preset
- Vitest：`vitest.config.ts` 搭配 test globals
- pytest：`pytest.ini` 或 `pyproject.toml` 节

**4. 验证设置：**
```bash
# 运行空测试套件——应以 0 个测试通过
npm test  # Node
pytest    # Python
go test ./...  # Go
cargo test    # Rust
```

**5. 创建第一个测试文件：**
遵循项目惯例的测试位置：
- 与源文件相邻的 `*.test.ts` / `*.spec.ts`
- `__tests__/` 目录
- 根目录下的 `tests/` 目录

框架设置是一次性成本，包含在第一个 TDD 计划的 RED 阶段中。
</framework_setup>

<error_handling>
## 错误处理

**RED 阶段测试未失败：**
- 功能可能已存在——调查
- 测试可能错误（未测试你预期的内容）
- 先修复再继续

**GREEN 阶段测试未通过：**
- 调试实现
- 不要跳到重构
- 持续迭代直到绿灯

**REFACTOR 阶段测试失败：**
- 撤销重构
- 提交为时过早
- 以更小的步骤重构

**无关测试失效：**
- 停止并调查
- 可能表明耦合问题
- 先修复再继续
</error_handling>

<commit_pattern>
## TDD 计划提交模式

TDD 计划产出 2-3 个原子 commit（每个阶段一个）：

```
test(08-02): add failing test for email validation

- Tests valid email formats accepted
- Tests invalid formats rejected
- Tests empty input handling

feat(08-02): implement email validation

- Regex pattern matches RFC 5322
- Returns boolean for validity
- Handles edge cases (empty, null)

refactor(08-02): extract regex to constant (optional)

- Moved pattern to EMAIL_REGEX constant
- No behavior changes
- Tests still pass
```
</commit_pattern>

<gate_enforcement>
## 门强制执行规则

当配置中启用 `workflow.tdd_mode` 时，对所有 `type: tdd` 计划强制执行 RED/GREEN/REFACTOR 门序列。

### 门定义

| 门 | 必需 | 提交模式 | 验证 |
|------|----------|---------------|------------|
| RED | 是 | `test({phase}-{plan}): ...` | 测试存在且在实现前失败 |
| GREEN | 是 | `feat({phase}-{plan}): ...` | 测试在实现后通过 |
| REFACTOR | 否 | `refactor({phase}-{plan}): ...` | 清理后测试仍然通过 |

### 快速失败规则

1. **RED 阶段意外 GREEN：** 如果在任何实现代码编写之前测试就通过了，停止。功能可能已存在或测试错误。先调查再继续。
2. **缺少 RED commit：** 如果没有 `test(...)` commit 在 `feat(...)` commit 之前，TDD 原则被违反。在 SUMMARY.md 中标记。
3. **REFACTOR 破坏测试：** 立即撤销重构。提交为时过早——以更小的步骤重构。
</gate_enforcement>

<end_of_phase_review>
## 阶段结束 TDD 审查检查点

当启用 `workflow.tdd_mode` 时，execute-phase 编排器在所有 wave 完成后、阶段验证前插入协作审查检查点。
这是咨询性的——不阻止阶段完成，但呈现 TDD 原则问题供人工审查。
</end_of_phase_review>

<context_budget>
## 上下文预算

TDD 计划目标约 **40% 上下文使用量**（低于标准计划的约 50%）。

更低的理由：
- RED 阶段：编写测试、运行测试、潜在地调试为什么未失败
- GREEN 阶段：实现、运行测试、潜在地迭代修复
- REFACTOR 阶段：修改代码、运行测试、验证无回归

每个阶段涉及读取文件、运行命令、分析输出。来回操作本质上比线性任务执行更重。

单一功能聚焦确保整个循环中的完整质量。
</context_budget>
