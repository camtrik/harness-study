# CLAUDE.md 模板

项目根目录 `CLAUDE.md` 的模板——由 `gsd-tools generate-claude-md` 自动生成。

包含 7 个由标记限定的段。各段可独立更新。
`generate-claude-md` 子命令管理 6 个段（project、stack、conventions、architecture、skills、workflow enforcement）。
profile 段由 `generate-claude-profile` 独家管理。

---

## 段模板

### Project 段
```
<!-- GSD:project-start source:PROJECT.md -->
## Project

{{project_content}}
<!-- GSD:project-end -->
```

**回退文本：**
```
项目尚未初始化。运行 /gsd-new-project 进行设置。
```

### Stack 段
```
<!-- GSD:stack-start source:STACK.md -->
## Technology Stack

{{stack_content}}
<!-- GSD:stack-end -->
```

**回退文本：**
```
技术栈尚未记录。将在代码库映射或首个阶段后填充。
```

### Conventions 段
```
<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

{{conventions_content}}
<!-- GSD:conventions-end -->
```

**回退文本：**
```
约定尚未建立。将在开发过程中随模式涌现而填充。
```

### Architecture 段
```
<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

{{architecture_content}}
<!-- GSD:architecture-end -->
```

**回退文本：**
```
架构尚未映射。请遵循代码库中已有的模式。
```

### Skills 段
```
<!-- GSD:skills-start source:skills/ -->
## Project Skills

| Skill          | Description           | Path                      |
| -------------- | --------------------- | ------------------------- |
| {{skill_name}} | {{skill_description}} | `{{skill_path}}/SKILL.md` |
<!-- GSD:skills-end -->
```

**回退文本：**
```
未找到项目 skill。将包含 `SKILL.md` 索引文件的 skill 添加到以下任一目录：`.claude/skills/`、`.agents/skills/`、`.cursor/skills/` 或 `.github/skills/`。
```

**发现行为：**
- 扫描 `.claude/skills/`、`.agents/skills/`、`.cursor/skills/`、`.github/skills/` 中是否包含 `SKILL.md` 的子目录
- 从 YAML frontmatter 中提取 `name` 和 `description`（支持多行描述）
- 跳过 GSD 自身安装的 skill（以 `gsd-` 开头的目录）
- 跨目录按 skill 名称去重

### Workflow Enforcement 段
```
<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

在使用 Edit、Write 或其他文件修改工具之前，请通过 GSD 命令开始工作，以确保规划产物和执行上下文保持同步。

使用以下入口点：
- `/gsd-quick` 用于小修复、文档更新和临时任务
- `/gsd-debug` 用于调查和 bug 修复
- `/gsd-execute-phase` 用于已规划的阶段工作

除非用户明确要求绕过，否则不要在 GSD 工作流之外直接编辑仓库。
<!-- GSD:workflow-end -->
```

### Profile 段（仅占位符）
```
<!-- GSD:profile-start -->
## Developer Profile

> 配置文件尚未配置。运行 `/gsd-profile-user` 生成您的开发者配置文件。
> 此段由 `generate-claude-profile` 管理——请勿手动编辑。
<!-- GSD:profile-end -->
```

**注意：** 此段不受 `generate-claude-md` 管理。它由 `generate-claude-profile` 独家管理。上面的占位符仅在创建新 CLAUDE.md 文件且不存在 profile 段时使用。

---

## 段排序

1. **Project** — 身份和目的（这个项目是什么）
2. **Stack** — 技术选择（使用了哪些工具）
3. **Conventions** — 代码模式和规则（代码如何编写）
4. **Architecture** — 系统结构（组件如何组合）
5. **Skills** — 已发现的项目 skill，含名称和描述（有哪些领域知识可用）
6. **Workflow Enforcement** — 文件修改类工作的默认 GSD 入口点
7. **Profile** — 开发者行为偏好（如何交互）

## 标记格式

- 开始：`<!-- GSD:{name}-start source:{file} -->`
- 结束：`<!-- GSD:{name}-end -->`
- source 属性支持在源文件变更时进行定向更新
- 开始标记的部分匹配（不含 `-->`）用于检测

## 回退行为

当源文件缺失时，回退文本提供 Claude 可执行的指导：
- 在缺乏数据时指导 Claude 的行为
- 不是占位广告或“缺失”通知
- 每个回退文本告诉 Claude 应该做什么，而不仅仅指出缺少什么
