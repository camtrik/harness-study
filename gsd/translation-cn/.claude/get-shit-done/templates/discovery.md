# 发现模板

`.planning/phases/XX-name/DISCOVERY.md` 的模板——用于库/方案决策的浅层研究。

**目的：** 在 plan-phase 的强制发现阶段，回答"应该用哪个库/方案"类问题。

对于深度生态系统研究（"专家是如何构建这个的"），请使用 `/gsd-plan-phase --research-phase`，它会生成 RESEARCH.md。

---

## 文件模板

```markdown
---
phase: XX-name
type: discovery
topic: [发现主题]
---

<session_initialization>
开始发现前，先验证今天的日期：
!`date +%Y-%m-%d`

在搜索"当前"或"最新"信息时使用此日期。
示例：如果今天是 2025-11-22，搜索"2025"而非"2024"。
</session_initialization>

<discovery_objective>
发现 [主题] 以支持 [阶段名称] 的实现。

目的：[此发现能支持什么决策/实现]
范围：[边界]
输出：含推荐方案的 DISCOVERY.md
</discovery_objective>

<discovery_scope>
<include>
- [需要回答的问题]
- [需要调查的领域]
- [需要的具体比较（如有）]
</include>

<exclude>
- [此发现的范围外]
- [延迟到实现阶段处理]
</exclude>
</discovery_scope>

<discovery_protocol>

**来源优先级：**
1. **Context7 MCP** - 用于库/框架文档（最新、权威）
2. **官方文档** - 用于平台特定或未索引的库
3. **WebSearch** - 用于比较、趋势、社区模式（验证所有发现）

**质量检查清单：**
在完成发现之前，验证：
- [ ] 所有声明都有权威来源（Context7 或官方文档）
- [ ] 否定声明（"X 不可能"）已通过官方文档验证
- [ ] API 语法/配置来自 Context7 或官方文档（绝不单独使用 WebSearch）
- [ ] WebSearch 发现已与权威来源交叉验证
- [ ] 最近的更新/changelog 已检查是否有破坏性变更
- [ ] 已考虑替代方法（不仅仅是找到的第一个方案）

**置信度级别：**
- HIGH：Context7 或官方文档已确认
- MEDIUM：WebSearch + Context7/官方文档已确认
- LOW：仅 WebSearch 或仅训练知识（标记待验证）

</discovery_protocol>


<output_structure>
创建 `.planning/phases/XX-name/DISCOVERY.md`：

```markdown
# [主题] 发现报告

## 摘要
[2-3 段执行摘要——研究了什么、发现了什么、推荐什么]

## 主要推荐
[做什么以及为什么——具体且可执行]

## 考虑过的替代方案
[还评估了哪些方案以及为什么没有选择]

## 关键发现

### [分类 1]
- [发现及其来源 URL 和与我们案例的相关性]

### [分类 2]
- [发现及其来源 URL 和相关性]

## 代码示例
[相关的实现模式，如适用]

## 元数据

<metadata>
<confidence level="high|medium|low">
[为什么是这个置信度级别——基于来源质量和验证]
</confidence>

<sources>
- [使用的主要权威来源]
</sources>

<open_questions>
[无法确定或需要在实现期间验证的事项]
</open_questions>

<validation_checkpoints>
[如果置信度为 LOW 或 MEDIUM，列出在实现期间需要验证的具体事项]
</validation_checkpoints>
</metadata>
```
</output_structure>

<success_criteria>
- 所有范围内问题均以权威来源回答
- 质量检查清单项已完成
- 有明确的主要推荐
- 低置信度发现已标记验证检查点
- 已准备好为 PLAN.md 创建提供信息
</success_criteria>

<guidelines>
**何时使用 discovery：**
- 技术选择不明确（库 A vs B）
- 不熟悉的集成需要最佳实践
- 需要 API/库调研
- 单个决策待定

**何时不应使用：**
- 已建立的模式（CRUD、使用已知库的认证）
- 实现细节（延迟到执行阶段）
- 可以从现有项目上下文中回答的问题

**何时改用 RESEARCH.md：**
- 细分/复杂领域（3D、游戏、音频、shader）
- 需要生态系统知识，不仅仅是库的选择
- "专家是如何构建这个的"类问题
- 对这些情况使用 `/gsd-plan-phase --research-phase`
</guidelines>
