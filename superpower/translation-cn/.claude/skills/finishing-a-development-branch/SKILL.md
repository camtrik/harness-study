---
name: finishing-a-development-branch
description: 当实施完成、所有测试通过且需要决定如何整合工作时使用——通过提供结构化的合并、PR 或清理选项来引导开发工作的收尾
---

# 完成功能分支

## 概述

通过提供清晰的选项并处理所选工作流来引导开发工作的收尾。

**核心原则：** 验证测试 → 检测环境 → 提供选项 → 执行选择 → 清理。

**开始时声明：** "我正在使用 finishing-a-development-branch skill 来完成此项工作。"

## 执行流程

### 第 1 步：验证测试

**在提供选项之前，验证测试通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个）。必须修复后才能完成：

[显示失败信息]

在测试通过之前无法进行合并/PR。
```

停止。不要进入第 2 步。

**如果测试通过：** 继续第 2 步。

### 第 2 步：检测环境

**在提供选项之前确定工作空间状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这会决定显示哪个菜单以及清理如何工作：

| 状态 | 菜单 | 清理 |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准 4 选项 | 无需清理 worktree |
| `GIT_DIR != GIT_COMMON`，命名分支 | 标准 4 选项 | 基于来源（见第 6 步） |
| `GIT_DIR != GIT_COMMON`，游离 HEAD | 精简 3 选项（无合并） | 无需清理（外部管理） |

### 第 3 步：确定基础分支

```bash
# 尝试常见的基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："此分支是从 main 分出来的——对吗？"

### 第 4 步：展示选项

**普通仓库和命名分支 worktree——精确展示这 4 个选项：**

```
实施完成。你想要做什么？

1. 本地合并回 <base-branch>
2. 推送并创建拉取请求
3. 保持分支原样（我稍后处理）
4. 放弃此项工作

选哪个？
```

**游离 HEAD——精确展示这 3 个选项：**

```
实施完成。你当前处于游离 HEAD（外部管理的工作空间）。

1. 推送为新分支并创建拉取请求
2. 保持原样（我稍后处理）
3. 放弃此项工作

选哪个？
```

**不要添加解释**——保持选项简洁。

### 第 5 步：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根路径以确保 CWD 安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并——在删除任何东西之前验证成功
git checkout <base-branch>
git pull
git merge <feature-branch>

# 在合并结果上验证测试
<test command>

# 仅在合并成功后：清理 worktree（第 6 步），然后删除分支
```

然后：清理 worktree（第 6 步），再删除分支：

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
<2-3 条要点说明变更内容>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

**不要清理 worktree**——用户需要保留它以便根据 PR 反馈进行迭代。

#### 选项 3：保持原样

报告："保留分支 <name>。Worktree 保留在 <path>。"

**不要清理 worktree。**

#### 选项 4：放弃

**首先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- Worktree 位于 <path>

输入 'discard' 以确认。
```

等待精确确认。

确认后：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理 worktree（第 6 步），再强制删除分支：
```bash
git branch -D <feature-branch>
```

### 第 6 步：清理工作空间

**仅对选项 1 和 4 执行。** 选项 2 和 3 始终保留 worktree。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：** 普通仓库，没有 worktree 需要清理。完成。

**如果 worktree 路径位于 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下：** Superpowers 创建了此 worktree——我们负责清理。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自愈：清理任何过期的注册信息
```

**否则：** 宿主机环境（运行框架）拥有此工作空间。不要移除它。如果你的平台提供了 workspace-exit 工具，使用它。否则，保持工作空间原样。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持原样 | - | - | 是 | - |
| 4. 放弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并有问题的代码，创建失败的 PR
- **修正：** 提供选项之前始终验证测试

**开放式问题**
- **问题：** "接下来做什么？"含糊不清
- **修正：** 精确展示 4 个结构化选项（游离 HEAD 时展示 3 个）

**为选项 2 清理 worktree**
- **问题：** 移除了用户进行 PR 迭代所需的 worktree
- **修正：** 仅对选项 1 和 4 进行清理

**在移除 worktree 之前删除分支**
- **问题：** `git branch -d` 失败，因为 worktree 仍在引用该分支
- **修正：** 先合并，移除 worktree，然后删除分支

**在 worktree 内部运行 git worktree remove**
- **问题：** 当 CWD 位于要移除的 worktree 内部时，命令静默失败
- **修正：** 在执行 `git worktree remove` 之前，始终先 `cd` 到主仓库根目录

**清理运行框架拥有的 worktree**
- **问题：** 移除运行框架创建的 worktree 会导致幽灵状态
- **修正：** 仅清理位于 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下的 worktree

**放弃时无确认**
- **问题：** 意外删除工作成果
- **修正：** 要求输入 "discard" 以确认

## 红线

**绝不：**
- 在测试失败的情况下继续
- 未在合并结果上验证测试就合并
- 未经确认就删除工作成果
- 未经明确请求就强制推送
- 在确认合并成功之前移除 worktree
- 清理不是由你创建的 worktree（来源检查）
- 在 worktree 内部运行 `git worktree remove`

**始终：**
- 在提供选项之前验证测试
- 在展示菜单之前检测环境
- 精确展示 4 个选项（游离 HEAD 时展示 3 个）
- 选项 4 需要键入确认
- 仅对选项 1 和 4 清理 worktree
- 在移除 worktree 之前 `cd` 到主仓库根目录
- 移除后运行 `git worktree prune`
