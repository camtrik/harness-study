# Claude Code Skills 测试

使用 Claude Code CLI 对 superpowers skills 进行自动化测试。

## 概述

此测试套件用于验证 skills 是否正确加载以及 Claude 是否按预期遵循它们。测试通过无头模式 (`claude -p`) 调用 Claude Code 并验证其行为。

## 环境要求

- 已安装 Claude Code CLI 且位于 PATH 中（`claude --version` 应可正常运行）
- 已安装本地 superpowers 插件（安装方法见主 README）

## 运行测试

### 运行所有快速测试（推荐）：
```bash
./run-skill-tests.sh
```

### 运行集成测试（较慢，10-30 分钟）：
```bash
./run-skill-tests.sh --integration
```

### 运行指定测试：
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### 使用详细输出模式运行：
```bash
./run-skill-tests.sh --verbose
```

### 设置自定义超时时间：
```bash
./run-skill-tests.sh --timeout 1800  # 集成测试使用 30 分钟超时
```

## 测试结构

### test-helpers.sh
skills 测试的公共函数：
- `run_claude "prompt" [timeout]` - 使用 prompt 运行 Claude
- `assert_contains output pattern name` - 验证 pattern 存在
- `assert_not_contains output pattern name` - 验证 pattern 不存在
- `assert_count output pattern count name` - 验证匹配数量
- `assert_order output pattern_a pattern_b name` - 验证顺序
- `create_test_project` - 创建临时测试目录
- `create_test_plan project_dir` - 创建示例计划文件

### 测试文件

每个测试文件：
1. 引入 `test-helpers.sh`
2. 使用特定 prompt 运行 Claude Code
3. 使用断言验证预期行为
4. 成功返回 0，失败返回非零值

## 示例测试

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# 询问 Claude 关于该 skill 的问题
output=$(run_claude "What does the my-skill skill do?" 30)

# 验证响应
assert_contains "$output" "expected behavior" "Skill describes behavior"

echo "=== All tests passed ==="
```

## 当前测试

### 快速测试（默认运行）

#### test-subagent-driven-development.sh
测试 skill 内容和要求（约 2 分钟）：
- Skill 加载和可访问性
- 工作流顺序（spec 合规性检查先于代码质量检查）
- 自审查要求已文档化
- 计划读取效率已文档化
- spec 合规性审查员的怀疑态度已文档化
- 审查循环已文档化
- 任务上下文提供已文档化

### 集成测试（使用 --integration 标志）

#### test-subagent-driven-development-integration.sh
完整工作流执行测试（约 10-30 分钟）：
- 使用 Node.js 环境创建真实测试项目
- 创建包含 2 个任务的实现计划
- 使用 subagent-driven-development 执行计划
- 验证实际行为：
  - 计划在开始时读取一次（而非每个任务读取一次）
  - subagent prompt 中包含完整的任务文本
  - subagent 在报告前执行自审查
  - spec 合规性审查在代码质量检查之前进行
  - spec 审查员独立阅读代码
  - 产出可运行的实现
  - 测试通过
  - 创建正确的 git 提交

**测试内容：**
- 工作流端到端实际可用
- 我们的改进实际生效
- subagent 正确遵循 skill
- 最终代码功能正常且经过测试

#### test-requesting-code-review.sh
代码审查 subagent 的行为测试（约 5 分钟）：
- 构建一个带有基线提交的小型项目
- 添加第二个提交，植入两个真实 bug（SQL 注入、明文密码处理）
- 通过 requesting-code-review skill 派遣代码审查员
- 验证审查员将植入的 bug 标记为 Critical/Important 级别并拒绝批准

**测试内容：**
- skill 实际派遣了可正常工作的代码审查 subagent
- 审查员模板产出的审查员能够捕获明显的安全 bug
- 审查员不阿谀奉承——不会批准包含植入的 Critical 问题的 diff

## 添加新测试

1. 创建新的测试文件：`test-<skill-name>.sh`
2. 引入 test-helpers.sh
3. 使用 `run_claude` 和断言编写测试
4. 将测试文件添加到 `run-skill-tests.sh` 的测试列表中
5. 设置为可执行：`chmod +x test-<skill-name>.sh`

## 超时注意事项

- 默认超时：每个测试 5 分钟
- Claude Code 响应可能需要时间
- 如有需要使用 `--timeout` 进行调整
- 测试应聚焦以避免运行时间过长

## 调试失败的测试

使用 `--verbose` 可查看完整的 Claude 输出：
```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

不使用 verbose 时，仅失败测试显示输出。

## CI/CD 集成

在 CI 中运行：
```bash
# 为 CI 环境设置显式超时
./run-skill-tests.sh --timeout 900

# 退出码 0 = 成功，非零 = 失败
```

## 注意事项

- 测试验证的是 skill 的 *指令*，而非完整执行
- 完整的工作流测试将非常耗时
- 聚焦于验证关键的 skill 要求
- 测试应当具有确定性
- 避免测试实现细节
