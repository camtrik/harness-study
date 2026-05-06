---
name: gsd:map-codebase
description: 使用并行 mapper agent 分析代码库，生成 .planning/codebase/ 文档
argument-hint: "[--fast [--focus tech|arch|quality|concerns]] [--query <term>|status|diff|refresh] [area]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---

<objective>
使用并行的 gsd-codebase-mapper agent 分析现有代码库，生成结构化的代码库文档。

每个 mapper agent 探索一个焦点领域，并**直接将文档写入** `.planning/codebase/`。编排器仅接收确认，保持上下文使用最小化。

输出：.planning/codebase/ 文件夹，包含关于代码库状态的 7 个结构化文档。
</objective>

<execution_context>
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/map-codebase.md
</execution_context>

<flags>
- **--fast**：轻量级扫描模式 — 启动一个而非四个 mapper agent。接受可选的 `--focus` 值：`tech`、`arch`、`quality`、`concerns` 或 `tech+arch`（默认）。比完整地图更快且更低上下文。
- **--query**：代码库智能查询模式。子命令：`query <term>`、`status`、`diff`、`refresh`。需要配置中启用 intel（`intel.enabled: true`）。query/status/diff 内联运行；refresh 启动一个 agent。
- **（无标志）**：完整并行地图 — 启动 4 个 mapper agent 生成所有 7 个代码库文档。
</flags>

<context>
参数：$ARGUMENTS

解析 $ARGUMENTS 的第一个 token：
- 如果是 `--fast`：去除该标志，运行 scan 工作流（传递剩余参数，包括可选的 --focus）。
- 如果是 `--query`：去除该标志，运行 intel 工作流（传递剩余参数作为子命令）。
- 否则：将整个 $ARGUMENTS 作为焦点区域传递给 map-codebase 工作流。

**如果存在则加载项目状态：**
检查 .planning/STATE.md - 如果项目已初始化则加载上下文

**此命令可以在以下情况下运行：**
- /gsd-new-project 之前（存量代码库）- 首先创建代码库地图
- /gsd-new-project 之后（新建代码库）- 随代码演进而更新代码库地图
- 随时刷新代码库理解
</context>

<when_to_use>
**以下情况使用 map-codebase：**
- 初始化之前的存量项目（首先理解现有代码）
- 重大变更后刷新代码库地图
- 上手接触不熟悉的代码库
- 进行重大重构之前（理解当前状态）
- STATE.md 引用过时的代码库信息时

**以下情况跳过 map-codebase：**
- 还没有代码的新建项目（没有什么可以映射的）
- 简单的代码库（<5 个文件）
</when_to_use>

<process>
1. 检查 .planning/codebase/ 是否已存在（提供刷新或跳过选项）
2. 创建 .planning/codebase/ 目录结构
3. 启动 4 个并行的 gsd-codebase-mapper agent：
   - Agent 1：tech 焦点 → 写入 STACK.md、INTEGRATIONS.md
   - Agent 2：arch 焦点 → 写入 ARCHITECTURE.md、STRUCTURE.md
   - Agent 3：quality 焦点 → 写入 CONVENTIONS.md、TESTING.md
   - Agent 4：concerns 焦点 → 写入 CONCERNS.md
4. 等待 agent 完成，收集确认（而非文档内容）
5. 验证所有 7 个文档是否存在并显示行数
6. 提交代码库地图
7. 提供后续步骤（通常为：/gsd-new-project 或 /gsd-plan-phase）
</process>

<success_criteria>
- [ ] .planning/codebase/ 目录已创建
- [ ] mapper agent 已写入所有 7 个代码库文档
- [ ] 文档遵循模板结构
- [ ] 并行 agent 完成且无错误
- [ ] 用户知晓后续步骤
</success_criteria>
