<purpose>
自主驱动里程碑中的阶段 — 所有剩余阶段、通过 `--from N`/`--to N` 指定范围、或通过 `--only N` 指定单个阶段。对于每个未完成的阶段：使用 Skill() 扁平调用进行讨论 → 规划 → 执行。仅在需要明确的用户决策时暂停（灰色区域接受、阻塞项、验证请求）。每个阶段后重新读取 ROADMAP.md 以捕获动态插入的阶段。
</purpose>

<required_reading>
在开始之前，阅读调用 prompt 的 execution_context 中引用的所有文件。
</required_reading>

<process>

<step name="initialize" priority="first">
## 1. 初始化

解析 `$ARGUMENTS` 中的 `--from N`、`--to N`、`--only N` 和 `--interactive` 标志。通过里程碑级 init 引导：

```bash
INIT=$(gsd-sdk query init.milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

解析 JSON 获取里程碑和阶段计数。

如果 `roadmap_exists` 为 false：错误 — 先运行 `/gsd-new-milestone`。
如果 `state_exists` 为 false：错误 — 先运行 `/gsd-new-milestone`。

显示启动横幅，包含里程碑版本信息和模式（单阶段/范围/交互式）。
</step>

<step name="discover_phases">
## 2. 发现阶段

运行阶段发现，过滤到未完成阶段，应用 --from/--to/--only 过滤器，按编号升序排序。如果没有未完成阶段剩余，显示完成消息并干净退出。提取每个阶段的名称、目标和成功标准。
</step>

<step name="execute_phase">
## 3. 执行阶段

显示进度横幅。然后：

**3a. 智能讨论** — 检查 CONTEXT.md 是否已存在。如果存在且 discuss 未禁用，则跳过。如果 `workflow.skip_discuss` 为 true，生成最小化 CONTEXT.md。否则执行智能讨论步骤。

**3a.5. UI 设计契约（前端阶段）** — 检查前端指标和 UI-SPEC 存在性，如果适用则生成 UI-SPEC。

**3b. 规划** — 运行 plan-phase（交互模式下作为后台 agent）。

**3c. 执行** — 运行 execute-phase（交互模式下作为后台 agent）。

**3c.5. 代码审查和修复** — 如果已启用，自动调用代码审查和修复链。

**3d. 执行后路由** — 读取验证状态并根据 passed/human_needed/gaps_found 分支处理。包括 gap 闭合重试（限制 1 次）。

**3d.5. UI 审查（前端阶段）** — 如果阶段有 UI-SPEC，运行 UI 审查审计（非阻塞，仅供参考）。
</step>

<step name="smart_discuss">
## 智能讨论

> 完整说明在 `get-shit-done/references/autonomous-smart-discuss.md` 中。现在读取该文件并严格遵循。

智能讨论是 gsd-discuss-phase 的自主优化变体。它以批处理表格形式提出灰色区域答案。
</step>

<step name="iterate">
## 4. 迭代

在每个阶段完成后重新读取 ROADMAP.md 以捕获中间插入的阶段。重新过滤未完成阶段。为交互式模式启用流水线并行。如果所有阶段完成，进入生命周期步骤。
</step>

<step name="lifecycle">
## 5. 生命周期

在所有阶段完成后，运行里程碑生命周期序列：审计 → 完成 → 清理。包括审计结果路由（passed → 自动继续，gaps_found → 用户决定，tech_debt → 用户决定）。
</step>

<step name="handle_blocker">
## 6. 处理阻塞项

当任何阶段操作失败或检测到阻塞项时，提供 3 个选项：修复并重试、跳过此阶段、停止自主模式。并显示进度摘要。
</step>

</process>

<success_criteria>
- [ ] 所有未完成阶段按顺序执行
- [ ] 智能讨论在表格中提出灰色区域答案
- [ ] 阶段之间显示进度横幅
- [ ] 执行后验证读取 VERIFICATION.md 并根据状态路由
- [ ] Gap 闭合限制为 1 次重试（防止无限循环）
- [ ] 计划阶段和执行阶段失败路由到 handle_blocker
- [ ] 每个阶段后重新读取 ROADMAP.md
- [ ] 每个阶段前检查 STATE.md 中的阻塞项
- [ ] 前端阶段在规划前生成 UI-SPEC
- [ ] 前端阶段在成功执行后进行 UI 审查审计
- [ ] --only N 限制执行恰好一个阶段，跳过生命周期
- [ ] --to N 在阶段 N 完成后停止执行
- [ ] --interactive 启用流水线并行
</success_criteria>
