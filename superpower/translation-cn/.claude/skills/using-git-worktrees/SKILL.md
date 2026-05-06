---
name: using-git-worktrees
description: 在开始需要与当前工作区隔离的功能工作时，或在执行实现计划之前使用——通过原生工具或 git worktree 回退方案确保隔离工作区存在
---

# 使用 Git Worktree

## 概述

确保工作在隔离的工作区中进行。优先使用你平台的原生工作树工具。仅在没有原生工具可用时才回退到手动 git worktree。

**核心原则：** 先检测已有隔离。然后用原生工具。然后回退到 git。永远不要和运行框架对着干。

**开始时声明：** "我正在使用 using-git-worktrees skill 来设置隔离工作区。"

## 第 0 步：检测已有隔离

**在创建任何东西之前，检查你是否已在隔离工作区中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块防护：** `GIT_DIR != GIT_COMMON` 在 git 子模块内部也会为真。在得出"已在工作树中"的结论之前，验证你不在子模块中：

```bash
# 如果返回路径，你在子模块中，而不是工作树——按普通仓库对待
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`（且不在子模块中）：** 你已经在链接的工作树中。跳到第 3 步（项目设置）。不要创建另一个工作树。

报告分支状态：
- 在分支上："已在隔离工作区 `<path>`，在分支 `<name>` 上。"
- 分离 HEAD："已在隔离工作区 `<path>`（分离 HEAD，外部管理）。完成时需要创建分支。"

**如果 `GIT_DIR == GIT_COMMON`（或在子模块中）：** 你在普通的仓库检出中。

用户是否已在指令中声明了工作树偏好？如果没有，在创建工作树之前征求同意：

> "你希望我设置一个隔离工作树吗？它可以保护你当前分支免受修改影响。"

尊重任何已有的明确偏好，无需询问。如果用户拒绝同意，直接在原地工作，跳到第 3 步。

## 第 1 步：创建隔离工作区

**你有两种机制。按以下顺序尝试。**

### 1a. 原生工作树工具（首选）

用户已要求隔离工作区（第 0 步同意）。你是否已有创建工作树的方法？它可能是一个名叫 `EnterWorktree`、`WorktreeCreate` 的工具，或是一个 `/worktree` 命令，或是一个 `--worktree` 标志。如果有，使用它并跳到第 3 步。

原生工具会自动处理目录放置、分支创建和清理。当你有原生工具时使用 `git worktree add` 会产生你的运行框架看不到也无法管理的幽灵状态。

仅当你没有原生工作树工具可用时，才继续执行第 1b 步。

### 1b. Git Worktree 回退方案

**仅在第 1a 步不适用时使用此方案**——你没有可用的原生工作树工具。使用 git 手动创建工作树。

#### 目录选择

按以下优先顺序执行。用户明确偏好始终优先于观察到的文件系统状态。

1. **检查你的指令中是否有声明的工作树目录偏好。** 如果用户已经指定了一个，直接使用，无需询问。

2. **检查是否存在项目本地的工作树目录：**
   ```bash
   ls -d .worktrees 2>/dev/null     # 首选（隐藏）
   ls -d worktrees 2>/dev/null      # 备选
   ```
   如果找到，使用它。两者都存在时，`.worktrees` 优先。

3. **检查是否存在全局目录：**
   ```bash
   project=$(basename "$(git rev-parse --show-toplevel)")
   ls -d ~/.config/superpowers/worktrees/$project 2>/dev/null
   ```
   如果找到，使用它（与旧的全局路径向后兼容）。

4. **如果没有其他指导可用**，默认使用项目根目录下的 `.worktrees/`。

#### 安全性验证（仅项目本地目录）

**创建工作树之前必须验证目录已被忽略：**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：** 添加到 .gitignore，提交更改，然后继续。

**为什么至关重要：** 防止意外将工作树内容提交到仓库。

全局目录（`~/.config/superpowers/worktrees/`）无需验证。

#### 创建工作树

```bash
project=$(basename "$(git rev-parse --show-toplevel)")

# 根据选择的位置确定路径
# 对于项目本地：path="$LOCATION/$BRANCH_NAME"
# 对于全局：path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退方案：** 如果 `git worktree add` 因权限错误（沙箱拒绝）而失败，告诉用户沙箱阻止了工作树创建，你将改为在当前目录工作。然后在原地运行设置和基线测试。

## 第 3 步：项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 第 4 步：验证干净基线

运行测试以确保工作区干净启动：

```bash
# 使用适合项目的命令
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：** 报告失败，询问是否继续或调查。

**如果测试通过：** 报告就绪。

### 报告

```
工作树就绪于 <full-path>
测试通过（<N> 个测试，0 个失败）
准备实现 <feature-name>
```

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| 已在链接的工作树中 | 跳过创建（第 0 步） |
| 在子模块中 | 按普通仓库对待（第 0 步防护） |
| 原生工作树工具可用 | 使用它（第 1a 步） |
| 无原生工具 | Git worktree 回退方案（第 1b 步） |
| `.worktrees/` 存在 | 使用它（验证已被忽略） |
| `worktrees/` 存在 | 使用它（验证已被忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 检查指令文件，然后默认 `.worktrees/` |
| 全局路径存在 | 使用它（向后兼容） |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 创建时权限错误 | 沙箱回退方案，原地工作 |
| 基线测试失败 | 报告失败 + 询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 和运行框架对着干

- **问题：** 在平台已提供隔离时使用 `git worktree add`
- **修复：** 第 0 步检测已有隔离。第 1a 步优先使用原生工具。

### 跳过检测

- **问题：** 在已有工作树内部创建嵌套工作树
- **修复：** 在创建任何东西之前始终运行第 0 步

### 跳过忽略验证

- **问题：** 工作树内容被跟踪，污染 git 状态
- **修复：** 在创建项目本地工作树之前始终使用 `git check-ignore`

### 擅自假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：** 遵循优先级：已有 > 全局旧版 > 指令文件 > 默认

### 测试失败时继续

- **问题：** 无法区分新 bug 和已存在的问题
- **修复：** 报告失败，获得明确许可以后再继续

## 危险信号

**绝不：**
- 在第 0 步检测到已有隔离时创建工作树
- 当你有原生工作树工具（例如 `EnterWorktree`）时使用 `git worktree add`。这是头号错误——如果你有它，就用它。
- 跳过第 1a 步直接使用第 1b 步的 git 命令
- 在未验证目录已被忽略的情况下创建工作树（项目本地）
- 跳过基线测试验证
- 测试失败时不询问就继续

**始终：**
- 先运行第 0 步检测
- 优先使用原生工具而非 git 回退方案
- 遵循目录优先级：已有 > 全局旧版 > 指令文件 > 默认
- 验证项目本地目录已被忽略
- 自动检测并运行项目设置
- 验证干净测试基线
