<planning_config>

`.planning/` 目录行为的配置选项。

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "base_branch": null,
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}",
  "quick_branch_template": null
},
"manager": {
  "flags": {
    "discuss": "",
    "plan": "",
    "execute": ""
  }
}
```

| 选项 | 默认值 | 描述 |
|--------|---------|-------------|
| `commit_docs` | `true` | 是否将规划产物提交到 git |
| `search_gitignored` | `false` | 向广泛的 rg 搜索添加 `--no-ignore` |
| `git.branching_strategy` | `"none"` | Git 分支方法：`"none"`、`"phase"` 或 `"milestone"` |
| `git.base_branch` | `null`（自动检测） | PR 和合并的目标分支（例如 `"master"`、`"develop"`）。当 `null` 时，从 `git symbolic-ref refs/remotes/origin/HEAD` 自动检测，回退为 `"main"`。 |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | phase 策略的分支模板 |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | milestone 策略的分支模板 |
| `git.quick_branch_template` | `null` | 快速任务运行的可选分支模板 |
| `workflow.use_worktrees` | `true` | executor agent 是否在隔离的 git worktree 中运行。设为 `false` 以禁用 worktree——agent 在主工作树上顺序执行。推荐个人开发者或当 worktree 合并引起问题时使用。 |
| `workflow.subagent_timeout` | `300000` | 并行子 agent 任务的超时（毫秒）（例如代码库映射）。对大型代码库或较慢的模型增加。默认：300000（5 分钟）。 |
| `workflow.inline_plan_threshold` | `2` | 不超过此数量的任务的计划内联执行（模式 C）而非生成子 agent。避免小计划的约 14K token 生成开销。设为 `0` 以始终生成子 agent。 |
| `manager.flags.discuss` | `""` | 从 manager 分发时传递给 `/gsd-discuss-phase` 的标志（例如 `"--auto --analyze"`） |
| `manager.flags.plan` | `""` | 从 manager 分发时传递给 plan 工作流的标志 |
| `manager.flags.execute` | `""` | 从 manager 分发时传递给 execute 工作流的标志 |
| `response_language` | `null` | 所有阶段/子 agent 中面向用户的问题和提示的语言（例如 `"Portuguese"`、`"Japanese"`、`"Spanish"`）。设置后，所有生成的 agent 包含以此语言响应的指令。 |
</config_schema>

<commit_docs_behavior>

**当 `commit_docs: true`（默认）时：**
- 规划文件正常提交
- SUMMARY.md、STATE.md、ROADMAP.md 在 git 中跟踪
- 保留规划决策的完整历史

**当 `commit_docs: false` 时：**
- 跳过 `.planning/` 文件的所有 `git add`/`git commit`
- 用户必须将 `.planning/` 添加到 `.gitignore`
- 适用于：OSS 贡献、客户项目、保持规划私密

**使用 `gsd-sdk query`（推荐）：**

```bash
# 自动 commit_docs + gitignore 检查的提交：
gsd-sdk query commit "docs: update state" --files .planning/STATE.md

# 通过 state load 加载配置（返回 JSON）：
INIT=$(gsd-sdk query state.load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs 在 JSON 输出中可用

# 或使用包含 commit_docs 的 init 命令：
INIT=$(gsd-sdk query init.execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs 包含在所有 init 命令的输出中
```

**自动检测：** 如果 `.planning/` 被 gitignore，`commit_docs` 自动为 `false`，无视 config.json 中的值。这防止了当用户有 `.planning/` 在 `.gitignore` 中时出现 git 错误。

**通过 CLI 提交（自动处理检查）：**

```bash
gsd-sdk query commit "docs: update state" --files .planning/STATE.md
```

CLI 在内部检查 `commit_docs` 配置和 gitignore 状态——无需手动条件判断。

</commit_docs_behavior>

<search_behavior>

**当 `search_gitignored: false`（默认）时：**
- 标准 rg 行为（遵循 .gitignore）
- 直接路径搜索有效：`rg "pattern" .planning/` 找到文件
- 广泛搜索跳过 gitignored：`rg "pattern"` 跳过 `.planning/`

**当 `search_gitignored: true` 时：**
- 对应该包含 `.planning/` 的广泛 rg 搜索添加 `--no-ignore`
- 仅在搜索整个仓库且期望 `.planning/` 匹配时才需要

**注意：** 大多数 GSD 操作使用直接文件读取或显式路径，这与 gitignore 状态无关。

</search_behavior>

<setup_uncommitted_mode>

使用未提交模式：

1. **设置配置：**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **添加到 .gitignore：**
   ```
   .planning/
   ```

3. **已跟踪的现有文件：** 如果 `.planning/` 之前被跟踪过：
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

4. **分支合并：** 当使用 `branching_strategy: phase` 或 `milestone` 时，`complete-milestone` 工作流在 `commit_docs: false` 时自动从合并提交前的暂存区中剥离 `.planning/` 文件。

</setup_uncommitted_mode>

<branching_strategy_behavior>

**分支策略：**

| 策略 | 分支创建时间 | 分支范围 | 合并点 |
|----------|---------------------|--------------|-------------|
| `none` | 不创建 | N/A | N/A |
| `phase` | 在 `execute-phase` 开始时 | 单个阶段 | 用户在阶段后合并 |
| `milestone` | 在里程碑的第一个 `execute-phase` | 整个里程碑 | 在 `complete-milestone` 时 |

**当 `git.branching_strategy: "none"`（默认）时：**
- 所有工作提交到当前分支
- 标准 GSD 行为

**当 `git.branching_strategy: "phase"` 时：**
- `execute-phase` 在执行前创建/切换到分支
- 分支名来自 `phase_branch_template`（例如 `gsd/phase-03-authentication`）
- 所有计划提交都到该分支
- 用户手动在阶段完成后合并分支
- `complete-milestone` 提供合并所有阶段分支的选项

**当 `git.branching_strategy: "milestone"` 时：**
- 里程碑的第一个 `execute-phase` 创建里程碑分支
- 分支名来自 `milestone_branch_template`（例如 `gsd/v1.0-mvp`）
- 里程碑中的所有阶段提交到同一分支
- `complete-milestone` 提供将里程碑分支合并到 main 的选项

**模板变量：**

| 变量 | 可用位置 | 描述 |
|----------|--------------|-------------|
| `{phase}` | phase_branch_template | 零填充的阶段号（例如 "03"） |
| `{slug}` | 两者 | 小写、连字符名称 |
| `{milestone}` | milestone_branch_template | 里程碑版本（例如 "v1.0"） |

**分支创建：**

```bash
# phase 策略
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# milestone 策略
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**complete-milestone 的合并选项：**

| 选项 | Git 命令 | 结果 |
|--------|-------------|--------|
| Squash merge（推荐） | `git merge --squash` | 每个分支一个干净的 commit |
| 带历史合并 | `git merge --no-ff` | 保留所有单独的 commit |
| 不合并删除 | `git branch -D` | 丢弃分支工作 |
| 保留分支 | （无） | 稍后手动处理 |

推荐 squash merge——保持主分支历史干净，同时在分支中保留完整的开发历史（直到删除）。

**用例：**

| 策略 | 最适合 |
|----------|----------|
| `none` | 个人开发，简单项目 |
| `phase` | 按阶段代码审查，细粒度回滚，团队协作 |
| `milestone` | 发布分支，暂存环境，按版本 PR |

</branching_strategy_behavior>

<complete_field_reference>

## 完整的字段参考

从 `CONFIG_DEFAULTS`（core.cjs）和 `VALID_CONFIG_KEYS`（config.cjs）生成。

### 核心字段

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `model_profile` | string | `"balanced"` | `"quality"`、`"balanced"`、`"budget"`、`"inherit"` | 子 agent 的模型选择预设 |
| `mode` | string | `"interactive"` | `"interactive"`、`"yolo"` | 操作模式：`"interactive"` 显示门和确认；`"yolo"` 自主运行，无提示 |
| `granularity` | string | （无） | `"coarse"`、`"standard"`、`"fine"` | 阶段计划的规划深度（从已弃用的 `depth` 迁移） |
| `commit_docs` | boolean | `true` | `true`、`false` | 将 .planning/ 产物提交到 git（如果 .planning/ 被 gitignore 则自动为 false） |
| `search_gitignored` | boolean | `false` | `true`、`false` | 通过 `--no-ignore` 在广泛 rg 搜索中包含 gitignored 路径 |
| `phase_naming` | string | `"sequential"` | `"sequential"`、`"custom"` | 阶段编号：自动递增或任意字符串 ID |
| `project_code` | string\|null | `null` | 任何短字符串 | 阶段目录的前缀（例如 `"CK"` 产生 `CK-01-foundation`） |
| `response_language` | string\|null | `null` | 任何语言名称 | 面向用户提示的语言（例如 `"Portuguese"`、`"Japanese"`） |
| `context_window` | number | `200000` | `200000`、`1000000` | 上下文窗口大小；为 1M 上下文模型设置 `1000000` |
| `resolve_model_ids` | boolean\|string | `false` | `false`、`true`、`"omit"` | 将模型别名映射到完整 Claude ID；`"omit"` 返回空字符串 |
| `context` | string\|null | `null` | `"dev"`、`"research"`、`"review"` | 执行上下文配置文件，调整 agent 行为 |
| `review.models.<cli>` | string\|null | `null` | 任何模型 ID 字符串 | /gsd-review 的按 CLI 模型覆盖（例如 `review.models.gemini`）。当 null 时回退到 CLI 默认值。 |

### 工作流字段

通过 `workflow.*` 命名空间设置（例如 `"workflow": { "research": true }`）。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `workflow.research` | boolean | `true` | `true`、`false` | 规划前运行研究 agent |
| `workflow.plan_check` | boolean | `true` | `true`、`false` | 运行 plan-checker agent 验证计划 |
| `workflow.verifier` | boolean | `true` | `true`、`false` | 执行后运行 verifier agent |
| `workflow.nyquist_validation` | boolean | `true` | `true`、`false` | 启用 Nyquist 启发验证门 |
| `workflow.auto_prune_state` | boolean | `false` | `true`、`false` | 阶段完成时自动修剪旧的 STATE.md 条目（保留最近 3 个阶段） |
| `workflow.auto_advance` | boolean | `false` | `true`、`false` | 完成后自动前进到下一阶段 |
| `workflow.node_repair` | boolean | `true` | `true`、`false` | 尝试自动修复失败的计划节点 |
| `workflow.node_repair_budget` | number | `2` | 任何正整数 | 每个失败节点的最大修复重试次数 |
| `workflow.ai_integration_phase` | boolean | `true` | `true`、`false` | 在规划 AI 系统阶段之前运行 /gsd-ai-integration-phase |
| `workflow.ui_phase` | boolean | `true` | `true`、`false` | 为前端阶段生成 UI-SPEC.md |
| `workflow.ui_safety_gate` | boolean | `true` | `true`、`false` | 要求 UI 变更的安全门审批 |
| `workflow.text_mode` | boolean | `false` | `true`、`false` | 使用纯文本编号列表代替 AskUserQuestion 菜单 |
| `workflow.research_before_questions` | boolean | `false` | `true`、`false` | 在讨论阶段交互式问题之前运行研究 |
| `workflow.discuss_mode` | string | `"discuss"` | `"discuss"`、`"assumptions"` | discuss-phase 的默认模式 |
| `workflow.skip_discuss` | boolean | `false` | `true`、`false` | 完全跳过讨论阶段 |
| `workflow.use_worktrees` | boolean | `true` | `true`、`false` | 在隔离的 git worktree 中运行 executor agent |
| `workflow.subagent_timeout` | number | `300000` | 任何正整数（ms） | 并行子 agent 任务的超时（默认：5 分钟） |
| `workflow.inline_plan_threshold` | number | `2` | `0`–`10` | 任务不超过 N 个的计划内联执行，而非生成子 agent |
| `workflow.code_review` | boolean | `true` | `true`、`false` | 在 ship 工作流中启用内置代码审查步骤 |
| `workflow.code_review_depth` | string | `"standard"` | `"light"`、`"standard"`、`"deep"` | ship 工作流中代码审查分析的深度级别 |
| `workflow._auto_chain_active` | boolean | `false` | `true`、`false` | 内部：跟踪是否活跃的自主链入 |
| `workflow.security_enforcement` | boolean | `true` | `true`、`false` | 通过 `/gsd-secure-phase` 启用威胁模型锚定的安全验证 |
| `workflow.security_asvs_level` | number | `1` | `1`、`2`、`3` | OWASP ASVS 验证级别 |
| `workflow.security_block_on` | string | `"high"` | `"high"`、`"medium"`、`"low"` | 阻止阶段推进的最低严重性 |
| `workflow.post_planning_gaps` | boolean | `true` | `true`、`false` | 规划后差距报告（#2493） |

### Git 字段

通过 `git.*` 命名空间设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `git.branching_strategy` | string | `"none"` | `"none"`、`"phase"`、`"milestone"` | 阶段/里程碑隔离的 Git 分支方法 |
| `git.base_branch` | string\|null | `null`（自动检测） | 任何分支名 | PR 和合并的目标分支 |
| `git.phase_branch_template` | string | `"gsd/phase-{phase}-{slug}"` | 带 `{phase}`、`{slug}` 的模板 | `phase` 策略的分支命名模板 |
| `git.milestone_branch_template` | string | `"gsd/{milestone}-{slug}"` | 带 `{milestone}`、`{slug}` 的模板 | `milestone` 策略的分支命名模板 |
| `git.quick_branch_template` | string\|null | `null` | 带 `{slug}` 的模板 | 快速任务运行的可选分支模板 |

### 搜索和 API 字段

切换外部搜索集成。在项目创建时检测到 API key 时自动设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `brave_search` | boolean | `false` | `true`、`false` | 为研究 agent 启用 Brave web 搜索 |
| `firecrawl` | boolean | `false` | `true`、`false` | 启用 Firecrawl 页面抓取 |
| `exa_search` | boolean | `false` | `true`、`false` | 启用 Exa 语义搜索 |

### 功能字段

通过 `features.*` 命名空间设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `features.thinking_partner` | boolean | `false` | `true`、`false` | 在工作流决策点启用条件式扩展思考 |
| `features.global_learnings` | boolean | `false` | `true`、`false` | 启用将全局学习注入到 agent prompt 中 |

### Hook 字段

通过 `hooks.*` 命名空间设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `hooks.context_warnings` | boolean | `true` | `true`、`false` | 超过上下文预算时显示警告 |

### 学习字段

通过 `learnings.*` 命名空间设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `learnings.max_inject` | number | `10` | 任何正整数 | 每个会话注入到 agent prompt 中的最大全局学习条目数 |

### 情报字段

通过 `intel.*` 命名空间设置。控制 `/gsd-intel` 消费的可查询代码库情报系统。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `intel.enabled` | boolean | `false` | `true`、`false` | 启用可查询代码库情报系统 |

### Manager 字段

通过 `manager.*` 命名空间设置。

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `manager.flags.discuss` | string | `""` | 任何 CLI 标志字符串 | 从 manager 传递给 `/gsd-discuss-phase` 的标志 |
| `manager.flags.plan` | string | `""` | 任何 CLI 标志字符串 | 从 manager 传递给 plan 工作流的标志 |
| `manager.flags.execute` | string | `""` | 任何 CLI 标志字符串 | 从 manager 传递给 execute 工作流的标志 |

### 高级字段

| 键 | 类型 | 默认值 | 允许的值 | 描述 |
|-----|------|---------|----------------|-------------|
| `parallelization` | boolean\|object | `true` | `true`、`false`、`{ "enabled": true }` | 启用并行 wave 执行 |
| `model_overrides` | object\|null | `null` | `{ "<agent-type>": "<model-id>" }` | 按 agent 类型覆盖模型选择 |
| `agent_skills` | object | `{}` | `{ "<agent-type>": "<skill-set>" }` | 将技能集分配给特定 agent 类型 |
| `sub_repos` | array | `[]` | 相对路径字符串数组 | 拥有独立 `.git` 仓库的子目录 |

---

## 字段交互

几个配置字段互相影响或触发特殊行为：

1. **`commit_docs` 自动检测**——当 config.json 中未设置显式值且 `.planning/` 在 `.gitignore` 中时，`commit_docs` 自动解析为 `false`。config 中的显式 `true` 或 `false` 始终覆盖自动检测。

2. **`branching_strategy` 控制分支模板**——`phase_branch_template` 和 `milestone_branch_template` 字段仅在 `branching_strategy` 设置为 `"phase"` 或 `"milestone"` 时使用。

3. **`context_window` 阈值触发**——当 `context_window >= 500000` 时，工作流启用自适应上下文增强。

4. **`parallelization` 多态**——同时接受简单布尔值和带 `enabled` 字段的对象。

5. **搜索 API key 和标志**——`brave_search`、`firecrawl` 和 `exa_search` 在项目创建时如果检测到对应的 API key 则自动设为 `true`。

---

## 示例配置

### 最小——个人开发者

```json
{
  "model_profile": "balanced",
  "commit_docs": true,
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "use_worktrees": false
  }
}
```

### 带分支的团队项目

```json
{
  "model_profile": "quality",
  "commit_docs": true,
  "project_code": "APP",
  "git": {
    "branching_strategy": "phase",
    "base_branch": "develop",
    "phase_branch_template": "gsd/phase-{phase}-{slug}"
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true,
    "use_worktrees": true,
    "discuss_mode": "discuss"
  },
  "manager": {
    "flags": {
      "discuss": "",
      "plan": "",
      "execute": ""
    }
  },
  "response_language": "English"
}
```

### 大型代码库——1M 上下文及延长超时

```json
{
  "model_profile": "quality",
  "context_window": 1000000,
  "commit_docs": true,
  "project_code": "MEGA",
  "phase_naming": "sequential",
  "git": {
    "branching_strategy": "milestone",
    "milestone_branch_template": "gsd/{milestone}-{slug}"
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true,
    "subagent_timeout": 600000,
    "use_worktrees": true,
    "node_repair": true,
    "node_repair_budget": 3,
    "auto_advance": true
  },
  "brave_search": true,
  "hooks": {
    "context_warnings": true
  }
}
```

</complete_field_reference>

</planning_config>
