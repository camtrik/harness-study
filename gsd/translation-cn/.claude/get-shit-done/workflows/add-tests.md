<purpose>
为已完成的阶段生成单元测试和端到端测试，基于其 SUMMARY.md、CONTEXT.md 和实现。将每个变更文件分类为 TDD（单元测试）、E2E（浏览器测试）或跳过的类别，向用户呈现测试计划以供批准，然后按照 RED-GREEN 规范生成测试。

用户目前在每个阶段后手动编写 `/gsd-quick` prompt 来生成测试。此工作流通过适当的分类、质量门和差距报告来标准化这一过程。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="parse_arguments">
解析 `$ARGUMENTS` 中的：
- 阶段编号（整数、小数或字母后缀）→ 存储为 `$PHASE_ARG`
- 阶段编号后的剩余文本 → 存储为 `$EXTRA_INSTRUCTIONS`（可选）

示例：`/gsd-add-tests 12 focus on edge cases` → `$PHASE_ARG=12`，`$EXTRA_INSTRUCTIONS="focus on edge cases"`

如果未提供阶段参数：

```
ERROR: Phase number required
Usage: /gsd-add-tests <phase> [additional instructions]
Example: /gsd-add-tests 12
Example: /gsd-add-tests 12 focus on edge cases in the pricing module
```

退出。
</step>

<step name="init_context">
加载阶段操作上下文：

```bash
INIT=$(gsd-sdk query init.phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

从 init JSON 中提取：`phase_dir`、`phase_number`、`phase_name`。

验证阶段目录是否存在。如果不存在：
```
ERROR: Phase directory not found for phase ${PHASE_ARG}
Ensure the phase exists in .planning/phases/
```
退出。

按优先级顺序读取阶段产物：
1. `${phase_dir}/*-SUMMARY.md` — 实现了什么，变更了哪些文件
2. `${phase_dir}/CONTEXT.md` — 验收标准、决策
3. `${phase_dir}/*-VERIFICATION.md` — 用户验证的场景（如果已完成 UAT）

如果不存在 SUMMARY.md：
```
ERROR: No SUMMARY.md found for phase ${PHASE_ARG}
This command works on completed phases. Run /gsd-execute-phase first.
```
退出。

呈现横幅：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ADD TESTS — Phase ${phase_number}: ${phase_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="analyze_implementation">
从 SUMMARY.md（"Files Changed" 或相应部分）中提取阶段修改的文件列表。

对每个文件进行三类分类：

| 分类 | 标准 | 测试类型 |
|----------|----------|-----------|
| **TDD** | 纯函数，可以编写 `expect(fn(input)).toBe(output)` | 单元测试 |
| **E2E** | 可通过浏览器自动化验证的 UI 行为 | Playwright/E2E 测试 |
| **Skip** | 无法有意义的测试或已被覆盖 | 无 |

**TDD 分类 — 适用于以下情况：**
- 业务逻辑：计算、定价、税务规则、验证
- 数据转换：映射、过滤、聚合、格式化
- 解析器：CSV、JSON、XML、自定义格式解析
- 验证器：输入验证、schema 验证、业务规则
- 状态机：状态转换、工作流步骤
- 工具函数：字符串处理、日期处理、数字格式化

**E2E 分类 — 适用于以下情况：**
- 键盘快捷键：键绑定、修饰键、和弦序列
- 导航：页面转换、路由、面包屑、前进/后退
- 表单交互：提交、验证错误、字段聚焦、自动补全
- 选择：行选择、多选、shift 点击范围
- 拖放：重新排序、在容器之间移动
- 模态对话框：打开、关闭、确认、取消
- 数据网格：排序、过滤、内联编辑、列大小调整

**Skip 分类 — 适用于以下情况：**
- UI 布局/样式：CSS 类、视觉外观、响应式断点
- 配置：配置文件、环境变量、特性标志
- 胶水代码：依赖注入设置、中间件注册、路由表
- 迁移：数据库迁移、schema 更改
- 简单 CRUD：无业务逻辑的基本创建/读取/更新/删除操作
- 类型定义：无逻辑的 record、DTO、接口

读取每个文件以验证分类。不要仅根据文件名分类。
</step>

<step name="present_classification">
将分类呈现给用户，在继续之前确认：

**文本模式（`workflow.text_mode: true` 在配置中或 `--text` 标志）：** 如果 `$ARGUMENTS` 中存在 `--text` 或 init JSON 中 `text_mode` 为 `true`，则设置 `TEXT_MODE=true`。当 TEXT_MODE 激活时，将每个 `AskUserQuestion` 调用替换为纯文本编号列表，并让用户输入其选择编号。这对于不支持 `AskUserQuestion` 的非 Claude 运行时（OpenAI Codex、Gemini CLI 等）是必需的。

```
AskUserQuestion(
  header: "Test Classification",
  question: |
    ## Files classified for testing

    ### TDD (Unit Tests) — {N} files
    {list of files with brief reason}

    ### E2E (Browser Tests) — {M} files
    {list of files with brief reason}

    ### Skip — {K} files
    {list of files with brief reason}

    {if $EXTRA_INSTRUCTIONS: "Additional instructions: ${EXTRA_INSTRUCTIONS}"}

    How would you like to proceed?
  options:
    - "Approve and generate test plan"
    - "Adjust classification (I'll specify changes)"
    - "Cancel"
)
```

如果用户选择 "Adjust classification"：应用其更改并重新呈现。
如果用户选择 "Cancel"：优雅退出。
</step>

<step name="discover_test_structure">
在生成测试计划之前，发现项目现有的测试结构：

```bash
# Find existing test directories
find . -type d -name "*test*" -o -name "*spec*" -o -name "*__tests__*" 2>/dev/null | head -20
# Find existing test files for convention matching
find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "*Tests.fs" -o -name "*Test.fs" \) 2>/dev/null | head -20
# Check for test runners
ls package.json *.sln 2>/dev/null || true
```

识别：
- 测试目录结构（单元测试在哪里，E2E 测试在哪里）
- 命名规范（`.test.ts`、`.spec.ts`、`*Tests.fs` 等）
- 测试运行器命令（如何执行单元测试，如何执行 E2E 测试）
- 测试框架（xUnit、NUnit、Jest、Playwright 等）

如果测试结构不明确，询问用户：
```
AskUserQuestion(
  header: "Test Structure",
  question: "I found multiple test locations. Where should I create tests?",
  options: [list discovered locations]
)
```
</step>

<step name="generate_test_plan">
为每个批准的文件创建详细的测试计划。

**对于 TDD 文件**，按照 RED-GREEN-REFACTOR 计划测试：
1. 识别文件中可测试的函数/方法
2. 对于每个函数：列出输入场景、预期输出、边界情况
3. 注意：由于代码已存在，测试可能立即通过 — 这没问题，但要验证它们测试的是正确的行为

**对于 E2E 文件**，按照 RED-GREEN 门计划测试：
1. 从 CONTEXT.md/VERIFICATION.md 中识别用户场景
2. 对于每个场景：描述用户操作、预期结果、断言
3. 注意：RED 门意味着确认如果功能出问题测试会失败

呈现完整的测试计划：

```
AskUserQuestion(
  header: "Test Plan",
  question: |
    ## Test Generation Plan

    ### Unit Tests ({N} tests across {M} files)
    {for each file: test file path, list of test cases}

    ### E2E Tests ({P} tests across {Q} files)
    {for each file: test file path, list of test scenarios}

    ### Test Commands
    - Unit: {discovered test command}
    - E2E: {discovered e2e command}

    Ready to generate?
  options:
    - "Generate all"
    - "Cherry-pick (I'll specify which)"
    - "Adjust plan"
)
```

如果 "Cherry-pick"：询问用户要包含哪些测试。
如果 "Adjust plan"：应用更改并重新呈现。
</step>

<step name="execute_tdd_generation">
对于每个批准的 TDD 测试：

1. **创建测试文件**，遵循发现的项目规范（目录、命名、导入）

2. **编写测试**，具有清晰的 arrange/act/assert 结构：
   ```
   // Arrange — set up inputs and expected outputs
   // Act — call the function under test
   // Assert — verify the output matches expectations
   ```

3. **运行测试**：
   ```bash
   {discovered test command}
   ```

4. **评估结果：**
   - **测试通过**：好的 — 实现满足测试。验证测试检查的是有意义的行为（而不仅仅是能编译）。
   - **测试失败，有断言错误**：这可能是测试发现的真正 bug。标记它：
     ```
     ⚠️ Potential bug found: {test name}
     Expected: {expected}
     Actual: {actual}
     File: {implementation file}
     ```
     不要修复实现 — 这是一个测试生成命令，不是修复命令。记录该发现。
   - **测试失败，有错误（导入、语法等）**：这是测试错误。修复测试并重新运行。
</step>

<step name="execute_e2e_generation">
对于每个批准的 E2E 测试：

1. **检查是否已有覆盖相同场景的测试**：
   ```bash
   grep -r "{scenario keyword}" {e2e test directory} 2>/dev/null || true
   ```
   如果找到了，扩展而不是重复。

2. **创建测试文件**，针对 CONTEXT.md/VERIFICATION.md 中的用户场景

3. **运行 E2E 测试**：
   ```bash
   {discovered e2e command}
   ```

4. **评估结果：**
   - **GREEN（通过）**：记录成功
   - **RED（失败）**：确定是测试问题还是真正的应用 bug。标记 bug：
     ```
     ⚠️ E2E failure: {test name}
     Scenario: {description}
     Error: {error message}
     ```
   - **无法运行**：报告阻塞。不要标记为完成。
     ```
     🛑 E2E blocker: {reason tests cannot run}
     ```

**不跳过规则：** 如果 E2E 测试无法执行（缺少依赖项、环境问题），报告阻塞并将测试标记为未完成。切勿在未实际运行测试的情况下标记成功。
</step>

<step name="summary_and_commit">
创建测试覆盖率报告并呈现给用户：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TEST GENERATION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Results

| Category | Generated | Passing | Failing | Blocked |
|----------|-----------|---------|---------|---------|
| Unit     | {N}       | {n1}    | {n2}    | {n3}    |
| E2E      | {M}       | {m1}    | {m2}    | {m3}    |

## Files Created/Modified
{list of test files with paths}

## Coverage Gaps
{areas that couldn't be tested and why}

## Bugs Discovered
{any assertion failures that indicate implementation bugs}
```

记录测试生成到项目状态中：
```bash
gsd-sdk query state-snapshot
```

如果有通过的测试需要提交：

```bash
git add {test files}
git commit -m "test(phase-${phase_number}): add unit and E2E tests from add-tests command"
```

呈现下一步：

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

{if bugs discovered:}
**Fix discovered bugs:** `/gsd-quick fix the {N} test failures discovered in phase ${phase_number}`

{if blocked tests:}
**Resolve test blockers:** {description of what's needed}

{otherwise:}
**All tests passing!** Phase ${phase_number} is fully tested.

---

**Also available:**
- `/gsd-add-tests {next_phase}` — test another phase
- `/gsd-verify-work {phase_number}` — run UAT verification

---
```
</step>

</process>

<success_criteria>
- [ ] Phase artifacts loaded (SUMMARY.md, CONTEXT.md, optionally VERIFICATION.md)
- [ ] All changed files classified into TDD/E2E/Skip categories
- [ ] Classification presented to user and approved
- [ ] Project test structure discovered (directories, conventions, runners)
- [ ] Test plan presented to user and approved
- [ ] TDD tests generated with arrange/act/assert structure
- [ ] E2E tests generated targeting user scenarios
- [ ] All tests executed — no untested tests marked as passing
- [ ] Bugs discovered by tests flagged (not fixed)
- [ ] Test files committed with proper message
- [ ] Coverage gaps documented
- [ ] Next steps presented to user
</success_criteria>
