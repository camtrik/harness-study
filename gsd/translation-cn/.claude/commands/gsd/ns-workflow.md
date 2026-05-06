---
name: gsd-workflow
description: "工作流 | discuss plan execute verify phase progress"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的阶段流水线 skill。
下列子 skill 名称是 #2790 之后合并的目标 — `gsd-phase`
吸收了之前的 add/insert/remove/edit-phase 命令，而 `gsd-progress`
吸收了之前的 next/do 命令。

| 用户想要 | 调用 |
|---|---|
| 规划前收集上下文 | gsd-discuss-phase |
| 澄清阶段交付内容 | gsd-spec-phase |
| 创建 PLAN.md | gsd-plan-phase |
| 执行阶段中的计划 | gsd-execute-phase |
| 通过 UAT 验证已构建的功能 | gsd-verify-work |
| 添加/插入/删除/编辑阶段 | gsd-phase |
| 推进到下一个逻辑步骤 | gsd-progress |
| 将规划卸载到 ultraplan 云 | gsd-ultraplan-phase |
| 跨 AI 计划审查收敛循环 | gsd-plan-review-convergence |

使用 Skill 工具直接调用匹配的 skill。
