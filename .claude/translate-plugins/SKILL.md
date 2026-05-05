---
name: translate-plugins
description: 翻译当前项目子文件夹中的插件配置目录（.claude、.codex、.cursor、.gemini 等）的所有内容到中文，方便用户学习。当用户说"翻译 gsd"、"翻译 superpower 的 .claude"、"翻译 superpower/.codex"、"把 gsd 翻译成中文"，或任何涉及翻译项目子文件夹下 AI 编程工具配置文件到中文的请求时触发。即使用户没有指定具体目录，也应该翻译该子文件夹下所有可翻译的配置目录。
---

# 翻译插件配置文件

将当前项目子文件夹中的 AI 编程工具配置目录翻译为地道的中文，方便学习和理解。

每个 AI 编程工具（Claude Code、Codex、Cursor、Gemini 等）都会在项目子文件夹中放置配置目录，里面包含 skills、agents、commands 等定义文件。这个 skill 的目的是把这些英文配置**完整翻译为中文**，输出到 `translation-cn/` 目录下，让中文用户可以直接阅读学习。

常见的配置目录：

| 目录 | 工具 |
|------|------|
| `.claude/` | Claude Code |
| `.codex/` | OpenAI Codex |
| `.cursor/` | Cursor |
| `.gemini/` | Google Gemini |
| `.omc/` | OMC |

这些目录下的典型内容：

```
配置目录/
├── agents/          ← agent 定义文件（.md）
├── skills/          ← skill 定义文件（SKILL.md + references/）
├── commands/        ← 命令定义文件
├── get-shit-done/   ← GSD 工作流配置（.md 文件）
├── hooks/           ← 钩子脚本（.js、.sh）
├── settings.json    ← 配置文件
└── ...
```

## 输入

用户指定要翻译的内容。可能的格式：

- 子文件夹名称：`gsd`、`superpower`（翻译该子文件夹下所有配置目录）
- 指定具体目录：`superpower/.claude`、`gsd/.codex`（只翻译这一个）
- 多个目标：`gsd .claude .codex`、`gsd, superpower`

## 翻译流程

### 1. 定位源目录

根据用户输入确定要翻译的源目录：

1. 如果用户给的是子文件夹名称（如 `gsd`），扫描该子文件夹下**所有**以 `.` 开头的配置目录（`.claude/`、`.codex/`、`.cursor/`、`.gemini/` 等），全部翻译
2. 如果用户给的是具体路径（如 `gsd/.codex`），只翻译该目录
3. 如果用户给的路径不存在，告诉用户并列出该子文件夹下实际存在的配置目录供选择

确认源目录存在后，用 `find` 列出其下**所有文件**（递归）。

### 2. 确定输出目录

输出规则：在子文件夹下创建 `translation-cn/`，然后保持原目录名。即：

```
源：  superpower/.claude/
输出：superpower/translation-cn/.claude/

源：  superpower/.codex/
输出：superpower/translation-cn/.codex/

源：  gsd/.claude/
输出：gsd/translation-cn/.claude/

源：  gsd/.gemini/
输出：gsd/translation-cn/.gemini/
```

如果一次翻译子文件夹下的所有配置目录，它们全部输出到同一个 `translation-cn/` 下：

```
gsd/.claude/    → gsd/translation-cn/.claude/
gsd/.codex/     → gsd/translation-cn/.codex/
gsd/.cursor/    → gsd/translation-cn/.cursor/
gsd/.gemini/    → gsd/translation-cn/.gemini/
```

### 3. 按文件类型处理

#### 需要翻译的文件（`.md`）

所有 `.md` 文件都进行**完整翻译**。

**必须翻译的内容：**
- 所有说明性文字、段落描述 → 译为自然流畅的中文
- 标题和章节名 → 译为中文
- 表格中的文字内容 → 译为中文
- 列表项中的说明文字 → 译为中文
- YAML frontmatter 中的 `description` 字段 → 译为中文
- 代码块内的注释 → 译为中文
- 提示语、警告语、注意事项 → 译为中文

**不翻译的内容：**
- 代码块内的代码逻辑（保持原样）
- 文件路径、命令行命令
- 专有名词：TDD、YAGNI、RED/GREEN/REFACTOR、mock、skill、subagent、agent、plugin 等
- 工具名称：npm、jest、vi、bash 等
- YAML frontmatter 中的 `name` 字段

#### 直接复制的文件（非 `.md`）

所有非 `.md` 文件不翻译，原样复制到输出目录，保持目录结构：
- 脚本文件：`.js`、`.ts`、`.sh`、`.cjs`、`.mjs`
- 配置文件：`.json`、`.toml`、`.yaml`、`.yml`
- 其他文件：`.html`、`.dot`、`VERSION` 等

### 4. 翻译风格

- 自然、流畅的中文技术文档风格——读起来像是中文母语者写的，而不是机器翻译的
- 保留原有的 markdown 结构和格式（标题层级、列表、表格、代码块等）
- Good/Bad 示例中的标签可译为"好"/"坏"或"推荐"/"不推荐"
- dot/graphviz 代码块保持原样

**翻译验证：** 翻译完成后，抽查输出文件的前几行，确认内容确实是中文。如果发现仍是英文，说明翻译没有生效，需要重新处理。

### 5. 保存翻译文件

将翻译后的文件保存到输出目录，保持原始的相对路径和文件名。例如：

```
输入（superpower/.claude/）：
  skills/test-driven-development/SKILL.md
  skills/test-driven-development/references/testing-anti-patterns.md
  hooks/gsd-prompt-guard.js
  settings.json

输入（superpower/.codex/）：
  skills/brainstorming/SKILL.md
  agents/planner.md

输出（superpower/translation-cn/）：
  .claude/skills/test-driven-development/SKILL.md              ← 翻译
  .claude/skills/test-driven-development/references/...md      ← 翻译
  .claude/hooks/gsd-prompt-guard.js                            ← 原样复制
  .claude/settings.json                                        ← 原样复制
  .codex/skills/brainstorming/SKILL.md                         ← 翻译
  .codex/agents/planner.md                                     ← 翻译
```

如果输出目录已存在同名文件，覆盖它们。

### 6. 完成汇报

翻译完成后，按配置目录分组汇报：

```
翻译完成。已在 superpower/translation-cn/ 下创建了 20 个文件：

.claude/（15 个文件）：
  翻译：agents/gsd-code-reviewer.md、skills/test-driven-development/SKILL.md、...（10 个 .md）
  复制：hooks/gsd-prompt-guard.js、settings.json、...（5 个）

.codex/（5 个文件）：
  翻译：skills/brainstorming/SKILL.md、agents/planner.md、...（4 个 .md）
  复制：config.toml（1 个）
```

## 批量翻译

如果用户一次提供多个子文件夹名称（如逗号分隔），对每个子文件夹按上述流程处理，可以并行读取不同文件夹的文件以加快速度。

如果用户说"翻译所有子文件夹"或类似的话，先列出当前项目下所有包含配置目录的子文件夹，确认后再逐一翻译。

## 翻译一致性

同一个项目下的多个文件会共享大量术语。翻译时保持以下术语一致性：

| 英文 | 中文 |
|------|------|
| skill | skill（不翻译） |
| human partner | 人类搭档 |
| subagent | subagent（不翻译） |
| agent | agent（不翻译） |
| plugin | plugin（不翻译） |
| harness | 运行框架 |
| mock | mock（不翻译） |
| refactoring | 重构 |
| edge case | 边界情况 |
| regression | 回归 |
| technical debt | 技术债务 |
| production code | 生产代码 |
| code review | 代码审查 |
| assertion | 断言 |
| benchmark | 基准测试 |
| prompt | prompt（不翻译） |
| template | 模板 |
| workflow | 工作流 |
| trigger | 触发 |
| orchestrator | 编排器 |
| phase | 阶段 |
| milestone | 里程碑 |
| roadmap | 路线图 |
| hook | hook（不翻译） |
| context window | 上下文窗口 |
| token | token（不翻译） |

如果遇到新的术语，选择最通用的中文翻译，并在心里记住保持一致。
