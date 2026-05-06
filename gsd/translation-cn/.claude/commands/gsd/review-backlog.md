---
name: gsd:review-backlog
description: 审查 backlog 项目并提升到活跃里程碑
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
审查所有 999.x backlog 项目并可选择性地将其提升到活跃
里程碑序列中，或删除过时条目。
</objective>

<process>

1. **列出 backlog 项目：**
   ```bash
   ls -d .planning/phases/999* 2>/dev/null || echo "未找到 backlog 项目"
   ```

2. **读取 ROADMAP.md** 并提取所有 999.x 阶段条目：
   ```bash
   cat .planning/ROADMAP.md
   ```
   显示每个 backlog 项目及其描述、任何累积的上下文（CONTEXT.md、RESEARCH.md）和创建日期。

3. **通过 AskUserQuestion 向用户展示列表：**
   - 对于每个 backlog 项目，显示：阶段编号、描述、累积的制品
   - 每个项目的选项：**提升**（移至活跃）、**保留**（留在 backlog）、**删除**

4. **对于要提升的项目：**
   - 在活跃里程碑中找到下一个顺序阶段编号
   - 将目录从 `999.x-slug` 重命名为 `{new_num}-slug`：
     ```bash
     NEW_NUM=$(gsd-sdk query phase.add "${DESCRIPTION}" --raw)
     ```
   - 将累积的制品移动到新的阶段目录
   - 更新 ROADMAP.md：将条目从 `## Backlog` 部分移动到活跃阶段列表
   - 移除 `(BACKLOG)` 标记
   - 添加适当的 `**Depends on:**` 字段

5. **对于要删除的项目：**
   - 删除阶段目录
   - 从 ROADMAP.md `## Backlog` 部分移除条目

6. **提交更改：**
   ```bash
   gsd-sdk query commit "docs: review backlog — promoted N, removed M" --files .planning/ROADMAP.md
   ```

7. **报告摘要：**
   ```
   ## 📋 Backlog Review Complete

   已提升：{已提升项目列表，带新阶段编号}
   已保留：{留在 backlog 中的项目列表}
   已删除：{已删除项目列表}
   ```

</process>
