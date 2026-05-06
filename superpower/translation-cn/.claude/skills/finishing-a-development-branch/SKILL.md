---
name: finishing-a-development-branch
description: 当实施完成、所有测试通过、你需要决定如何集成工作时使用——通过展示用于合并、PR 或清理的结构化选项来指导开发工作的完成
---

# 完成开发分支

## 概述

通过展示清晰的选项和处理所选工作流来指导开发工作的完成。

**核心原则：** 验证测试 → 检测环境 → 展示选项 → 执行选择 → 清理。

**开始时宣布：** "我正在使用 finishing-a-development-branch skill 来完成这项工作。"

## 流程

### 步骤 1：验证测试

**在展示选项之前，验证测试通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个失败）。必须在完成之前修复：

[显示失败]

在测试通过之前无法继续合并/PR。
```

停止。不要继续执行步骤 2。

**如果测试通过：** 继续执行步骤 2。

### 步骤 2：检测环境

**在展示选项之前确定工作区状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这决定了显示哪个菜单以及清理如何工作：

| 状态 | 菜单 | 清理 |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON`（正常仓库） | 标准 4 个选项 | 没有要清理的 worktree |
| `GIT_DIR != GIT_COMMON`、命名分支 | 标准 4 个选项 | 基于起源（见步骤 6） |
| `GIT_DIR != GIT_COMMON`、分离的 HEAD | 减少的 3 个选项（无合并） | 无清理（外部管理） |

### 步骤 3：确定基础分支

```bash
# 尝试常见的基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："此分支从 main 分离——这正确吗？"

### 步骤 4：展示选项

**正常仓库和命名分支 worktree——准确展示这 4 个选项：**

```
实施完成。你想做什么？

1. 在本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支原样（我稍后处理）
4. 丢弃这项工作

哪个选项？
```

**分离的 HEAD——准确展示这 3 个选项：**

```
实施完成。你处于分离的 HEAD（外部管理的工作区）。

1. 作为新分支推送并创建 Pull Request
2. 保持原样（我稍后处理）
3. 丢弃这项工作

哪个选项？
```

**不要添加解释**——保持选项简洁。

### 步骤 5：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根目录以确保 CWD 安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 首先合并——在删除任何内容之前验证成功
git checkout <base-branch>
git pull
git merge <feature-branch>

# 验证合并结果的测试
<test command>

# 只有在合并成功后：清理 worktree（步骤 6），然后删除分支
```

然后：清理 worktree（步骤 6），然后删除分支：

```bash
git branch -d <feature-branch>
```

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<2-3 个更改要点>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

**不要清理 worktree**——用户需要它来迭代 PR 反馈。

#### 选项 3：保持原样

报告： "保持分支 <name>。Worktree 保留在 <path>。"

**不要清理 worktree。**

#### 选项 4：丢弃

**首先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- <path> 处的 worktree

输入 'discard' 以确认。
```

等待确切的确认。

如果确认：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理 worktree（步骤 6），然后强制删除分支：
```bash
git branch -D <feature-branch>
```

### 步骤 6：清理工作区

**仅针对选项 1 和 4 运行。** 选项 2 和 3 始终保留 worktree。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：** 正常仓库，没有要清理的 worktree。完成。

**如果 worktree 路径在 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下：** Superpower 创建了此 worktree——我们拥有清理。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自愈：清理任何过时的注册
```

**否则：** 主机环境（运行框架）拥有此工作区。不要删除它。如果你的平台提供工作区退出工具，请使用它。否则，将工作区保留在原位。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持原样 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏的代码，创建失败的 PR
- **修复：** 在提供选项之前始终验证测试

**开放式问题**
- **问题：** "我接下来应该做什么？" 模糊不清
- **修复：** 准确展示 4 个结构化选项（或针对分离的 HEAD 展示 3 个）

**为选项 2 清理 worktree**
- **问题：** 删除用户进行 PR 迭代所需的 worktree
- **修复：** 仅针对选项 1 和 4 清理

**在删除分支之前删除 worktree**
- **问题：** `git branch -d` 失败，因为 worktree 仍引用该分支
- **修复：** 首先合并，删除 worktree，然后删除分支

**从 worktree 内部运行 git worktree remove**
- **问题：** 当 CWD 在正在删除的 worktree 内部时，命令静默失败
- **修复：** 在 `git worktree remove` 之前始终 `cd` 到主仓库根目录

**清理运行框架拥有的 worktree**
- **问题：** 删除运行框架创建的 worktree 会导致幻影状态
- **修复：** 仅清理 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下的 worktree

**丢弃时无确认**
- **问题：** 意外删除工作
- **修复：** 要求输入 "discard" 确认

## 危险信号

**永远不要：**
- 在测试失败时继续
- 在未验证结果测试的情况下合并
- 在没有确认的情况下删除工作
- 在没有明确请求的情况下强制推送
- 在确认合并成功之前删除 worktree
- 清理你没有创建的 worktree（起源检查）
- 从 worktree 内部运行 `git worktree remove`

**始终：**
- 在提供选项之前验证测试
- 在展示菜单之前检测环境
- 准确展示 4 个选项（或针对分离的 HEAD 展示 3 个）
- 获取选项 4 的输入确认
- 仅针对选项 1 和 4 清理 worktree
- 在 worktree 删除之前 `cd` 到主仓库根目录
- 在删除后运行 `git worktree prune`
