<overview>
GSD 框架的 Git 集成。
</overview>

<core_principle>

**提交产出，而非过程。**

git log 应该像已交付的变更日志，而不是规划活动的日记。
</core_principle>

<commit_points>

| 事件 | 提交？ | 原因 |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP 创建 | 是 | 项目初始化 |
| PLAN.md 创建 | 否 | 中间产物——与计划完成一起提交 |
| RESEARCH.md 创建 | 否 | 中间产物 |
| DISCOVERY.md 创建 | 否 | 中间产物 |
| **任务完成** | 是 | 原子工作单元（每个任务 1 个 commit） |
| **计划完成** | 是 | 元数据提交（SUMMARY + STATE + ROADMAP） |
| Handoff 创建 | 是 | 保留 WIP 状态 |

</commit_points>

<git_check>

```bash
[ -d .git ] && echo "GIT_EXISTS" || echo "NO_GIT"
```

如果 NO_GIT：静默运行 `git init`。GSD 项目始终拥有自己的仓库。
</git_check>

<commit_formats>

<format name="initialization">
## 项目初始化（brief + roadmap 一起）

```
docs: initialize [项目名称] ([N] 个阶段)

[来自 PROJECT.md 的一行摘要]

阶段：
1. [阶段名称]: [目标]
2. [阶段名称]: [目标]
3. [阶段名称]: [目标]
```

提交内容：

```bash
gsd-sdk query commit "docs: initialize [项目名称] ([N] 个阶段)" --files .planning/
```

</format>

<format name="task-completion">
## 任务完成（计划执行期间）

每个任务完成后立即提交。

> **并行 agent：** 当作为并行 executor 运行时（由 execute-phase 生成），
> 正常运行 commit——让 pre-commit hook 运行。默认情况下不要传递 `--no-verify`
> （#2924）。Hook 应在引入 commit 上触发；静默绕过违反项目
> CLAUDE.md 指导。如果项目通过
> `workflow.worktree_skip_hooks=true` 显式选择退出，编排器会在
> executor prompt 中显示该标志；缺少该信号时，hook 正常运行。

```
{类型}({阶段}-{计划}): {任务名称}

- [关键变更 1]
- [关键变更 2]
- [关键变更 3]
```

**提交类型：**
- `feat` - 新功能
- `fix` - Bug 修复
- `test` - 仅测试（TDD RED 阶段）
- `refactor` - 代码清理（TDD REFACTOR 阶段）
- `perf` - 性能改进
- `chore` - 依赖、配置、工具

**示例：**

```bash
# 标准任务
git add src/api/auth.ts src/types/user.ts
git commit -m "feat(08-02): create user registration endpoint

- POST /auth/register validates email and password
- Checks for duplicate users
- Returns JWT token on success
"

# TDD 任务——RED 阶段
git add src/__tests__/jwt.test.ts
git commit -m "test(07-02): add failing test for JWT generation

- Tests token contains user ID claim
- Tests token expires in 1 hour
- Tests signature verification
"

# TDD 任务——GREEN 阶段
git add src/utils/jwt.ts
git commit -m "feat(07-02): implement JWT generation

- Uses jose library for signing
- Includes user ID and expiry claims
- Signs with HS256 algorithm
"
```

</format>

<format name="plan-completion">
## 计划完成（所有任务完成后）

所有任务提交后，一个最终的元数据提交捕获计划完成。

```
docs({阶段}-{计划}): complete [计划名称] plan

已完成任务: [N]/[N]
- [任务 1 名称]
- [任务 2 名称]
- [任务 3 名称]

SUMMARY: .planning/phases/XX-name/{阶段}-{计划}-SUMMARY.md
```

提交内容：

```bash
gsd-sdk query commit "docs({阶段}-{计划}): complete [计划名称] plan" --files .planning/phases/XX-name/{阶段}-{计划}-PLAN.md .planning/phases/XX-name/{阶段}-{计划}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md
```

**注意：** 代码文件不包含——已按每个任务提交。

</format>

<format name="handoff">
## Handoff（WIP）

```
wip: [阶段名称] 暂停在任务 [X]/[Y]

当前: [任务名称]
[如果被阻塞:] 阻塞: [原因]
```

提交内容：

```bash
gsd-sdk query commit "wip: [阶段名称] 暂停在任务 [X]/[Y]" --files .planning/
```

</format>
</commit_formats>

<example_log>

**旧方法（按计划提交）：**
```
a7f2d1 feat(checkout): Stripe payments with webhook verification
3e9c4b feat(products): catalog with search, filters, and pagination
8a1b2c feat(auth): JWT with refresh rotation using jose
5c3d7e feat(foundation): Next.js 15 + Prisma + Tailwind scaffold
2f4a8d docs: initialize ecommerce-app (5 phases)
```

**新方法（按任务提交）：**
```
# 阶段 04 - 结账
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# 阶段 03 - 商品
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# 阶段 02 - 认证
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# 阶段 01 - 基础
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# 初始化
5c6d7e docs: initialize ecommerce-app (5 phases)
```

每个计划产出 2-4 个 commit（任务 + 元数据）。清晰、细粒度、可二分查找。

</example_log>

<anti_patterns>

**仍然不提交（中间产物）：**
- PLAN.md 创建（与计划完成一起提交）
- RESEARCH.md（中间产物）
- DISCOVERY.md（中间产物）
- 小的规划调整
- "修复了路线图中的拼写错误"

**要提交（产出）：**
- 每个任务完成（feat/fix/test/refactor）
- 计划完成元数据（docs）
- 项目初始化（docs）

**核心原则：** 提交工作代码和已交付的产出，而非规划过程。

</anti_patterns>

<commit_strategy_rationale>

## 为什么按任务提交？

**AI 的上下文工程：**
- Git 历史成为未来 Claude 会话的主要上下文来源
- `git log --grep="{阶段}-{计划}"` 显示一个计划的所有工作
- `git diff <hash>^..<hash>` 显示每个任务的确切变更
- 减少对解析 SUMMARY.md 的依赖 = 更多上下文用于实际工作

**失败恢复：**
- 任务 1 已提交 ✅，任务 2 失败 ❌
- Claude 在下一次会话：看到任务 1 完成，可以重试任务 2
- 可以 `git reset --hard` 到最后成功的任务

**调试：**
- `git bisect` 找到确切失败的任务，而不仅仅是失败的计划
- `git blame` 将行追溯到特定任务上下文
- 每个 commit 都可以独立回滚

**可观测性：**
- 个人开发者 + Claude 工作流受益于细粒度的归属
- 原子 commit 是 git 最佳实践
- 当消费者是 Claude 而非人类时，"提交噪音"无关紧要

</commit_strategy_rationale>

<sub_repos_support>

## 多仓库工作空间支持（sub_repos）

对于拥有独立 git 仓库的工作空间（例如 `backend/`、`frontend/`、`shared/`），GSD 将提交分别路由到每个仓库。

### 配置

在 `.planning/config.json` 中，在 `planning.sub_repos` 下列出子仓库目录：

```json
{
  "planning": {
    "commit_docs": false,
    "sub_repos": ["backend", "frontend", "shared"]
  }
}
```

设置 `commit_docs: false` 以便规划文档保留在本地且不提交到任何子仓库。

### 工作原理

1. **自动检测：** 在 `/gsd-new-project` 期间，拥有自身 `.git` 文件夹的目录会被检测并提供选择作为子仓库。在后续运行中，`loadConfig` 自动将 `sub_repos` 列表与文件系统同步——添加新创建的仓库并删除已删除的仓库。这意味着当磁盘上的仓库发生变化时，`config.json` 可能会被自动重写。
2. **文件分组：** 代码文件按其子仓库前缀分组（例如 `backend/src/api/users.ts` 属于 `backend/` 仓库）。
3. **独立提交：** 每个子仓库通过 `gsd-tools.cjs commit-to-subrepo` 接收其自身的原子 commit。文件路径在暂存前被设为相对于子仓库根目录。
4. **规划保留本地：** `.planning/` 目录不提交；它作为跨仓库协调。

### 提交路由

当配置了 `sub_repos` 时，使用 `commit-to-subrepo` 代替标准 `commit` 命令：

```bash
gsd-sdk query commit-to-subrepo "feat(02-01): add user API" \
  --files backend/src/api/users.ts backend/src/types/user.ts frontend/src/components/UserForm.tsx
```

这将 `src/api/users.ts` 和 `src/types/user.ts` 暂存到 `backend/` 仓库，将 `src/components/UserForm.tsx` 暂存到 `frontend/` 仓库，然后各自以相同的消息独立提交。

未匹配任何已配置子仓库的文件将被报告为未匹配。

</sub_repos_support>
