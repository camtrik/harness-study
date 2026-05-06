# Smart Discuss — 自主模式

Smart discuss 是 `gsd-discuss-phase` 的自主优化变体。它以批量表格的形式提出灰色区域答案——用户按区域接受或覆盖——然后写入与 discuss-phase 产生的完全相同的 CONTEXT.md。

**输入：** 来自 execute_phase 的 `PHASE_NUM`。运行 init 获取阶段路径：

```bash
PHASE_STATE=$(gsd-sdk query init.phase-op ${PHASE_NUM})
```

从 JSON 中解析：`phase_dir`、`phase_slug`、`padded_phase`、`phase_name`。

---

## 子步骤 1：加载先前上下文

读取项目级别和先前阶段的上下文，避免重复询问已决定的问题。

**读取项目文件：**

```bash
cat .planning/PROJECT.md 2>/dev/null || true
cat .planning/REQUIREMENTS.md 2>/dev/null || true
cat .planning/STATE.md 2>/dev/null || true
```

从中提取：
- **PROJECT.md**——愿景、原则、不可协商项、用户偏好
- **REQUIREMENTS.md**——验收标准、约束、必须有与最好有
- **STATE.md**——当前进度、迄今记录的决策

**读取所有先前的 CONTEXT.md 文件：**

```bash
(find .planning/phases -name "*-CONTEXT.md" 2>/dev/null || true) | sort
```

对于阶段号小于当前阶段的每个 CONTEXT.md：
- 读取 `<decisions>` 节——这些是锁定的偏好
- 读取 `<specifics>`——特定引用或"我想要像 X 那样"的时刻
- 注意模式（例如，"用户一致偏好极简 UI"、"用户拒绝了冗长的输出"）

**构建内部 prior_decisions 上下文**（不写入文件）：

```
<prior_decisions>
## 项目级别
- [来自 PROJECT.md 的关键原则或约束]
- [来自 REQUIREMENTS.md 的影响此阶段的需求]

## 来自先前的阶段
### 阶段 N：[名称]
- [与当前阶段相关的决策]
- [建立模式的偏好]
</prior_decisions>
```

如果没有先前上下文，继续即可——这在早期阶段是预期的。

---

## 子步骤 2：扫视代码库

轻量级的代码库扫描，为灰色区域识别和建议提供信息。保持在 ~5% 的上下文预算内。

**检查是否有现有的代码库映射：**

```bash
ls .planning/codebase/*.md 2>/dev/null || true
```

**如果存在代码库映射：** 读取最相关的那些（根据阶段类型确定 CONVENTIONS.md、STRUCTURE.md、STACK.md）。提取可复用组件、既定模式、集成点。跳到下面的构建上下文。

**如果没有代码库映射，进行有针对性的 grep：**

从阶段目标中提取关键术语。搜索相关文件：

```bash
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10 || true
ls src/components/ src/hooks/ src/lib/ src/utils/ 2>/dev/null || true
```

读取 3-5 个最相关的文件以理解现有模式。

**构建内部 codebase_context**（不写入文件）：
- **可复用资产**——在此阶段可用的现有组件、hook、工具
- **既定模式**——代码库如何进行状态管理、样式处理、数据获取
- **集成点**——新代码的连接位置（路由、导航、提供者）

---

## 子步骤 3：分析阶段并生成建议

**获取阶段详情：**

```bash
DETAIL=$(gsd-sdk query roadmap.get-phase ${PHASE_NUM})
```

从 JSON 响应中提取 `goal`、`requirements`、`success_criteria`。

**基础设施检测——在生成灰色区域之前首先检查：**

当以下所有条件都满足时，该阶段为纯基础设施：
1. 目标关键词匹配："scaffolding"、"plumbing"、"setup"、"configuration"、"migration"、"refactor"、"rename"、"restructure"、"upgrade"、"infrastructure"
2. 并且成功标准全部是技术性的："文件存在"、"测试通过"、"配置有效"、"命令运行"
3. 并且没有描述面向用户的行为（没有"用户可以"、"显示"、"展示"、"呈现")

**如果仅为基础架构：** 跳过子步骤 4。直接跳到子步骤 5，使用最小 CONTEXT.md。显示：

```
阶段 ${PHASE_NUM}：基础设施阶段——跳过讨论，写入最小上下文。
```

对 CONTEXT.md 使用以下默认值：
- `<domain>`：来自 ROADMAP 目标的阶段边界
- `<decisions>`：单个"### Claude 的自由裁量"子节——"所有实现选择由 Claude 自行决定——纯基础设施阶段"
- `<code_context>`：代码库扫视发现的任何内容
- `<specifics>`："没有具体要求——基础设施阶段"
- `<deferred>`："无"

**如果非基础设施——生成灰色区域建议：**

从阶段目标确定领域类型：
- 用户**看到**的东西 → 视觉：布局、交互、状态、密度
- 用户**调用**的东西 → 接口：合约、响应、错误、认证
- 用户**运行**的东西 → 执行：调用、输出、行为模式、标志
- 用户**阅读**的东西 → 内容：结构、语气、深度、流程
- 被**组织**的东西 → 组织：条件、分组、例外、命名

检查 prior_decisions——跳过先前阶段已决定的灰色区域。

生成 **3-4 个灰色区域**，每个区域 **~4 个问题**。对于每个问题：
- **预选择一个推荐答案**，基于：先前决策（一致性）、代码库模式（复用）、领域惯例（标准方法）、ROADMAP 成功标准
- 每个问题生成 **1-2 个替代方案**
- **标注**相关的先前决策上下文（"你在阶段 N 中决定了 X"）和代码上下文（"组件 Y 存在 Z 个变体"）

---

## 子步骤 4：按区域逐一呈现建议

**逐个**呈现灰色区域。对于每个区域（第 M/N 个）：

显示一个表格：

```
### 灰色区域 {M}/{N}：{区域名称}

| # | 问题 | ✅ 推荐 | 替代方案 |
|---|------|--------|---------|
| 1 | {问题} | {答案} — {理由} | {替代1}；{替代2} |
| 2 | {问题} | {答案} — {理由} | {替代1} |
| 3 | {问题} | {答案} — {理由} | {替代1}；{替代2} |
| 4 | {问题} | {答案} — {理由} | {替代1} |
```

然后通过 **AskUserQuestion** 提示用户：
- **header:** "区域 {M}/{N}"
- **question:** "为 {区域名称} 接受这些答案？"
- **options:** 动态构建——总是先"全部接受"，然后对每个问题（最多 4 个）分别设置"修改 Q1"到"修改 QN"，最后是"深入讨论"。最多明确列出 6 个选项（AskUserQuestion 会自动添加"Other"）。

**当选择"全部接受"：** 记录此区域的所有推荐答案。移到下一个区域。

**当选择"修改 QN"：** 使用 AskUserQuestion 针对该特定问题的替代选项：
- **header:** "{区域名称}"
- **question:** "Q{N}：{问题文本}"
- **options:** 列出 1-2 个替代方案加上"你决定"（映射为 Claude 的自由裁量）

记录用户的选择。重新显示包含更改的更新表格。重新呈现完整的接受提示，以便用户可以进行额外更改或接受。

**当选择"深入讨论"：** 仅针对此区域切换到交互模式——逐一提问，使用 AskUserQuestion，每个问题 2-3 个具体选项加上"你决定"。在 4 个问题后提示：
- **header:** "{区域名称}"
- **question:** "关于 {区域名称} 还有更多问题，还是移到下一个？"
- **options:** "更多问题" / "下一个区域"

如果选择"更多问题"，再问 4 个。如果选择"下一个区域"，显示此区域已捕获答案的最终汇总表并继续。

**当选择"Other"（自由文本）：** 解释为特定的变更请求或一般性反馈。纳入该区域的决策中，重新显示更新后的表格，重新呈现接受提示。

**范围蔓延处理：** 如果用户提到阶段领域之外的内容：

```
"{功能} 听起来像是一个新能力——它应该属于它自己的阶段。
我会将其记录为一个延迟想法。

回到 {当前区域}：{返回当前问题}"
```

在内部跟踪延迟想法，以便包含在 CONTEXT.md 中。

---

## 子步骤 5：写入 CONTEXT.md

所有区域解决后（或基础设施跳过），写入 CONTEXT.md 文件。

**文件路径：** `${phase_dir}/${padded_phase}-CONTEXT.md`

使用**完全**以下结构（与 discuss-phase 输出相同）：

```markdown
# 阶段 {PHASE_NUM}：{阶段名称} - 上下文

**收集时间：** {日期}
**状态：** 准备规划

<domain>
## 阶段边界

{来自分析的领域边界声明——此阶段交付什么}

</domain>

<decisions>
## 实现决策

### {区域 1 名称}
- {为 Q1 接受的/选择的答案}
- {为 Q2 接受的/选择的答案}
- {为 Q3 接受的/选择的答案}
- {为 Q4 接受的/选择的答案}

### {区域 2 名称}
- {为 Q1 接受的/选择的答案}
- {为 Q2 接受的/选择的答案}
...

### Claude 的自由裁量
{任何收集到的"你决定"答案——注明 Claude 在此处有灵活性}

</decisions>

<code_context>
## 现有代码洞察

### 可复用资产
- {来自代码库扫视——组件、hook、工具}

### 既定模式
- {来自代码库扫视——状态管理、样式处理、数据获取}

### 集成点
- {来自代码库扫视——新代码的连接位置}

</code_context>

<specifics>
## 具体想法

{来自讨论的任何特定引用或"我想要像 X 那样"}
{如果没有："没有具体要求——开放使用标准方法"}

</specifics>

<deferred>
## 延迟想法

{捕获的但超出此阶段范围的想法}
{如果没有："无——讨论保持在阶段范围内"}

</deferred>
```

写入文件。

**提交：**

```bash
gsd-sdk query commit "docs(${PADDED_PHASE}): smart discuss context" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

显示确认：

```
已创建：{路径}
已捕获决策：{数量} 个，跨越 {区域数量} 个区域
```
