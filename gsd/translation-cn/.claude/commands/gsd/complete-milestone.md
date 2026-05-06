---
type: prompt
name: gsd:complete-milestone
description: 归档已完成的里程碑并为下一版本做准备
argument-hint: <version>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
将里程碑 {{version}} 标记为完成，归档到 milestones/，并更新 ROADMAP.md 和 REQUIREMENTS.md。

目的：创建已交付版本的历史记录，归档里程碑制品（路线图和需求），为下一里程碑做准备。
输出：里程碑已归档（路线图 + 需求），PROJECT.md 已演进，已打 git 标签。
</objective>

<execution_context>
**立即加载以下文件（继续之前）：**

- @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/complete-milestone.md（主工作流）
- @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/templates/milestone-archive.md（归档模板）
  </execution_context>

<context>
**项目文件：**
- `.planning/ROADMAP.md`
- `.planning/REQUIREMENTS.md`
- `.planning/STATE.md`
- `.planning/PROJECT.md`

**用户输入：**

- 版本：{{version}}（例如 "1.0"、"1.1"、"2.0"）
  </context>

<process>

**遵循 complete-milestone.md 工作流：**

0. **检查审计：**

   - 查找 `.planning/v{{version}}-MILESTONE-AUDIT.md`
   - 如果缺失或过期：建议先运行 `/gsd-audit-milestone`
   - 如果审计状态为 `gaps_found`：建议内联关闭差距
     （审计输出已经列举了未满足的需求、跨阶段问题和损坏的流程——通过
     `/gsd-phase --insert <N>` 插入关闭阶段，加上标准的
     discuss/plan/execute 链条）再继续。
   - 如果审计状态为 `passed`：继续步骤 1

   ```markdown
   ## 预检

   {如果没有 v{{version}}-MILESTONE-AUDIT.md：}
   ⚠ 未找到里程碑审计。首先运行 `/gsd-audit-milestone` 来验证
   需求覆盖率、跨阶段集成和 E2E 流程。

   {如果审计存在差距：}
   ⚠ 里程碑审计发现差距。审计输出已经列举了
   未满足的需求、跨阶段问题和损坏的流程——为每个差距
   插入一个关闭阶段 `/gsd-phase --insert <N>` 并运行标准
   `/gsd-discuss-phase` → `/gsd-plan-phase` → `/gsd-execute-phase`
   链条。或者仍然继续，将差距作为技术债务接受。

   {如果审计通过：}
   ✓ 里程碑审计已通过。继续执行完成流程。
   ```

1. **验证就绪状态：**

   - 检查里程碑中所有阶段是否都有已完成的计划（SUMMARY.md 存在）
   - 展示里程碑范围和统计数据
   - 等待确认

2. **收集统计数据：**

   - 统计阶段、计划、任务数量
   - 计算 git 范围、文件变更、代码行数
   - 从 git log 提取时间线
   - 展示摘要，确认

3. **提取成果：**

   - 读取里程碑范围内所有阶段的 SUMMARY.md 文件
   - 提取 4-6 个关键成果
   - 展示以供审批

4. **归档里程碑：**

   - 创建 `.planning/milestones/v{{version}}-ROADMAP.md`
   - 从 ROADMAP.md 提取完整的阶段详情
   - 填充 milestone-archive.md 模板
   - 将 ROADMAP.md 更新为带链接的单行摘要

5. **归档需求：**

   - 创建 `.planning/milestones/v{{version}}-REQUIREMENTS.md`
   - 将所有 v1 需求标记为已完成（复选框打勾）
   - 注明需求结果（已验证、已调整、已放弃）
   - 删除 `.planning/REQUIREMENTS.md`（为下一里程碑创建新的）

6. **更新 PROJECT.md：**

   - 添加"当前状态"部分，包含已交付版本
   - 添加"下一个里程碑目标"部分
   - 将先前内容归档到 `<details>` 中（如果是 v1.1+）

7. **提交和打标签：**

   - 暂存：MILESTONES.md、PROJECT.md、ROADMAP.md、STATE.md、归档文件
   - 提交：`chore: archive v{{version}} milestone`
   - 标签：`git tag -a v{{version}} -m "[里程碑摘要]"`
   - 询问是否推送标签

8. **提供后续步骤：**
   - `/gsd-new-milestone` — 开始下一个里程碑（提问 → 研究 → 需求 → 路线图）

</process>

<success_criteria>

- 里程碑已归档到 `.planning/milestones/v{{version}}-ROADMAP.md`
- 需求已归档到 `.planning/milestones/v{{version}}-REQUIREMENTS.md`
- `.planning/REQUIREMENTS.md` 已删除（为下一里程碑提供全新文件）
- ROADMAP.md 已折叠为单行条目
- PROJECT.md 已更新当前状态
- 已创建 git 标签 v{{version}}
- 提交成功
- 用户知晓后续步骤（包括需要制定新需求）
  </success_criteria>

<critical_rules>

- **先加载工作流：** 执行前先读取 complete-milestone.md
- **验证完成情况：** 所有阶段必须有 SUMMARY.md 文件
- **用户确认：** 在验证关卡等待审批
- **删除前先归档：** 更新/删除原始文件前先创建归档文件
- **单行摘要：** ROADMAP.md 中折叠的里程碑应为包含链接的单行
- **上下文效率：** 归档保持 ROADMAP.md 和 REQUIREMENTS.md 每个里程碑的恒定大小
- **全新需求：** 下一里程碑以 `/gsd-new-milestone` 启动，其中包括需求定义
  </critical_rules>
