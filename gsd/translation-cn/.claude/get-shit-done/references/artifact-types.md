# GSD 产物类型

本参考文档记录 GSD 规划分类体系中的所有产物类型。每种类型都有定义的形态、生命周期、位置和消费机制。一个格式良好但没有工作流读取的产物是惰性的——消费机制才是赋予产物意义的关键。

---

## 核心产物

### ROADMAP.md
- **形态**：里程碑 + 阶段列表，包含目标和规范引用
- **生命周期**：创建 → 每个里程碑更新 → 归档
- **位置**：`.planning/ROADMAP.md`
- **消费方**：`plan-phase`、`discuss-phase`、`execute-phase`、`progress`、`state` 命令

### STATE.md
- **形态**：当前位置追踪器（阶段、计划、进度、决策）
- **生命周期**：整个项目期间持续更新
- **位置**：`.planning/STATE.md`
- **消费方**：所有编排工作流；`resume-project`、`progress`、`next` 命令

### REQUIREMENTS.md
- **形态**：编号的验收标准及可追溯性表
- **生命周期**：项目开始时创建 → 随着需求满足而更新
- **位置**：`.planning/REQUIREMENTS.md`
- **消费方**：`discuss-phase`、`plan-phase`、CONTEXT.md 生成；executor 标记完成

### CONTEXT.md（每阶段）
- **形态**：6 节格式：domain、decisions、canonical_refs、code_context、specifics、deferred
- **生命周期**：规划前创建 → 规划和执行期间使用 → 被下一阶段取代
- **位置**：`.planning/phases/XX-name/XX-CONTEXT.md`
- **消费方**：`plan-phase`（读取 decisions）、`execute-phase`（读取 code_context 和 canonical_refs）

### PLAN.md（每计划）
- **形态**：Frontmatter + objective + 含类型的 tasks + success criteria + output spec
- **生命周期**：planner 创建 → 执行 → 生成 SUMMARY.md
- **位置**：`.planning/phases/XX-name/XX-YY-PLAN.md`
- **消费方**：`execute-phase` executor；任务 commit 引用计划 ID

### SUMMARY.md（每计划）
- **形态**：含依赖图的 Frontmatter + 叙述 + deviations + self-check
- **生命周期**：计划完成时创建 → 被同阶段后续计划读取
- **位置**：`.planning/phases/XX-name/XX-YY-SUMMARY.md`
- **消费方**：编排器（progress）、planner（未来计划的上下文）、`milestone-summary`

### HANDOFF.json / .continue-here.md
- **形态**：结构化的暂停状态（JSON 机器可读 + Markdown 人类可读）
- **生命周期**：暂停时创建 → 恢复时消费 → 被下次暂停替换
- **位置**：`.planning/HANDOFF.json` + `.planning/phases/XX-name/.continue-here.md`（或 spike/deliberation 路径）
- **消费方**：`resume-project` 工作流

---

## 扩展产物

### DISCUSSION-LOG.md（每阶段）
- **形态**：discuss-phase 中假设和修正的审计追踪
- **生命周期**：讨论时创建 → 只读审计记录
- **位置**：`.planning/phases/XX-name/XX-DISCUSSION-LOG.md`
- **消费方**：人工审查；不被自动化工作流读取

### USER-PROFILE.md
- **形态**：校准层级和偏好配置
- **生命周期**：`profile-user` 创建 → 随着观察到的偏好更新
- **位置**：`/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/USER-PROFILE.md`
- **消费方**：`discuss-phase-assumptions`（校准层级）、`plan-phase`

### SPIKE.md / DESIGN.md（每 Spike）
- **形态**：研究问题 + 方法论 + 发现 + 建议
- **生命周期**：创建 → 调查 → 决策 → 归档
- **位置**：`.planning/spikes/SPIKE-NNN/`
- **消费方**：引用该 spike 的 planner；`pause-work` 用于 spike 上下文交接

### Spike README.md / MANIFEST.md（每 Spike，通过 /gsd-spike 创建）
- **形态**：YAML frontmatter（spike、name、validates、verdict、related、tags）+ 运行说明 + 结果
- **生命周期**：`/gsd-spike` 创建 → 验证 → `/gsd-spike-wrap-up` 完成
- **位置**：`.planning/spikes/NNN-name/README.md`、`.planning/spikes/MANIFEST.md`
- **消费方**：`/gsd-spike-wrap-up` 进行策展；`pause-work` 用于 spike 上下文交接

### Sketch README.md / MANIFEST.md / index.html（每 Sketch）
- **形态**：YAML frontmatter（sketch、name、question、winner、tags）+ 变体作为标签页 HTML
- **生命周期**：`/gsd-sketch` 创建 → 评估 → `/gsd-sketch-wrap-up` 完成
- **位置**：`.planning/sketches/NNN-name/README.md`、`.planning/sketches/NNN-name/index.html`、`.planning/sketches/MANIFEST.md`
- **消费方**：`/gsd-sketch-wrap-up` 进行策展；`pause-work` 用于 sketch 上下文交接

### WRAP-UP-SUMMARY.md（每次 wrap-up）
- **形态**：策展结果、包含/排除的项目、功能/设计区域分组
- **生命周期**：`/gsd-spike-wrap-up` 或 `/gsd-sketch-wrap-up` 创建
- **位置**：`.planning/spikes/WRAP-UP-SUMMARY.md` 或 `.planning/sketches/WRAP-UP-SUMMARY.md`
- **消费方**：项目历史；不被自动化工作流读取

---

## 常设参考产物

### METHODOLOGY.md

- **形态**：常设参考——可复用的解释性框架（镜头），可跨阶段应用
- **生命周期**：创建 → 活跃 → 被取代（当某个镜头被更好的替代）
- **位置**：`.planning/METHODOLOGY.md`（项目级别，非阶段级别）
- **内容**：命名的镜头，每个记录：
  - 诊断什么（它检测的问题类别）
  - 建议什么（它开出的响应类别）
  - 何时应用（触发条件）
  - 示例：贝叶斯更新、STRIDE 威胁建模、延迟成本优先级排序
- **消费方**：
  - `discuss-phase-assumptions`——读取 METHODOLOGY.md（如果存在）并对当前假设分析应用活跃镜头，再将发现呈现给用户
  - `plan-phase`——读取 METHODOLOGY.md 为每个计划提供方法论选择
  - `pause-work`——将 METHODOLOGY.md 包含在 `.continue-here.md` 的 Required Reading 节中，使恢复时的 agent 继承项目的分析思维取向

**为什么消费很重要：** 没有被工作流读取的 METHODOLOGY.md 是惰性的。镜头只有在 agent 将其加载到推理上下文中进行分析时才生效。这就是为什么 discuss-phase-assumptions 和 pause-work 工作流都显式引用此文件。

**镜头条目示例：**

```markdown
## 贝叶斯更新

**诊断：** 基于过时先验做出的决策——早期形成的假设已被后续证据驳斥，但仍嵌入计划中。

**建议：** 在确认假设之前，问："什么证据会让我改变这个？"如果没有任何证据能改变它，那就是信念而非假设。标记供用户审查。

**应用于：** 任何带有 Confident 标签但形成于近期架构变更、库升级或范围修正之前的假设时。
```
