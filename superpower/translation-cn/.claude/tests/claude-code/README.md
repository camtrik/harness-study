# Claude Code Skills 测试

使用 Claude Code CLI 对 superpower skills 进行自动化测试。

## 概述

此测试套件验证 skills 是否正确加载以及 Claude 是否按预期执行。测试以无头模式(`claude -p`)调用 Claude Code 并验证行为。

## 要求

- 已安装 Claude Code CLI 并在 PATH 中(`claude --version` 应该能工作)
- 已安装本地 superpowers plugin(参见主 README 了解安装说明)

## 运行测试

### 运行所有快速测试(推荐):
```bash
./run-skill-tests.sh
```

### 运行集成测试(较慢, 10-30 分钟):
```bash
./run-skill-tests.sh --integration
```

### 运行特定测试:
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### 使用详细输出运行:
```bash
./run-skill-tests.sh --verbose
```

### 设置自定义超时:
```bash
./run-skill-tests.sh --timeout 1800  # 集成测试 30 分钟
```

## 测试结构

### test-helpers.sh
skills 测试的通用函数:
- `run_claude "prompt" [timeout]` - 使用 prompt 运行 Claude
- `assert_contains output pattern name` - 验证模式存在
- `assert_not_contains output pattern name` - 验证模式不存在
- `assert_count output pattern count name` - 验证精确计数
- `assert_order output pattern_a pattern_b name` - 验证顺序
- `create_test_project` - 创建临时测试目录
- `create_test_plan project_dir` - 创建示例计划文件

### 测试文件

每个测试文件:
1. 引入 `test-helpers.sh`
2. 使用特定 prompts 运行 Claude Code
3. 使用断言验证预期行为
4. 成功返回 0,失败返回非零

## 测试示例

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# 询问 Claude 关于 skill 的情况
output=$(run_claude "What does the my-skill skill do?" 30)

# 验证响应
assert_contains "$output" "expected behavior" "Skill describes behavior"

echo "=== All tests passed ==="
```

## 当前测试

### 快速测试(默认运行)

#### test-subagent-driven-development.sh
测试 skill 内容和要求(~2 分钟):
- Skill 加载和可访问性
- 工作流排序(规范合规性在代码质量之前)
- 自我审查要求已记录
- 计划读取效率已记录
- 规范合规性审查者怀疑态度已记录
- 审查循环已记录
- 任务上下文提供已记录

### 集成测试(使用 --integration 标志)

#### test-subagent-driven-development-integration.sh
完整工作流执行测试(~10-30 分钟):
- 创建真实测试项目并设置 Node.js
- 创建包含 2 个任务的实施计划
- 使用 subagent-driven-development 执行计划
- 验证实际行为:
  - 计划在开始时读取一次(不是每个任务)
  - 完整任务文本在 subagent prompts 中提供
  - Subagents 在报告前执行自我审查
  - 规范合规性审查在代码质量之前发生
  - 规范审查者独立读取代码
  - 生成可工作的实现
  - 测试通过
  - 创建正确的 git commits

**它测试了什么:**
- 工作流端到端实际工作
- 我们的改进实际被应用
- Subagents 正确遵循 skill
- 最终代码功能正常且已测试

#### test-requesting-code-review.sh
代码审查者 subagent 的行为测试(~5 分钟):
- 构建一个带有基准提交的小型项目
- 添加第二个提交,植入两个真实 bug(SQL 注入、明文密码处理)
- 通过 requesting-code-review skill 分派代码审查者
- 验证审查者以 Critical/Important 严重性标记植入的 bug 并拒绝批准

**它测试了什么:**
- skill 实际分派了一个可工作的代码审查者 subagent
- 审查者模板生成的审查者能发现明显的安全 bug
- 审查者不是唯唯诺诺的 — 它不会批准有 Critical 问题的 diff

## 添加新测试

1. 创建新测试文件: `test-<skill-name>.sh`
2. 引入 test-helpers.sh
3. 使用 `run_claude` 和断言编写测试
4. 添加到 `run-skill-tests.sh` 中的测试列表
5. 设置可执行权限: `chmod +x test-<skill-name>.sh`

## 超时考虑

- 默认超时: 每个测试 5 分钟
- Claude Code 可能需要时间来响应
- 如需要使用 `--timeout` 调整
- 测试应该聚焦以避免长时间运行

## 调试失败的测试

使用 `--verbose`,你会看到完整的 Claude 输出:
```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

没有 verbose 时,只有失败会显示输出。

## CI/CD 集成

在 CI 中运行:
```bash
# 为 CI 环境使用显式超时运行
./run-skill-tests.sh --timeout 900

# 退出码 0 = 成功,非零 = 失败
```

## 注意

- 测试验证 skill *指令*,而不是完整执行
- 完整工作流测试会非常慢
- 聚焦于验证关键 skill 要求
- 测试应该是确定性的
- 避免测试实现细节
