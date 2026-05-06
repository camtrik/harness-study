# 上下文预算规则

保持编排器上下文精简的标准规则。在生成子 agent 或读取重要内容的工作流中引用此文档。

另请参阅：`references/universal-anti-patterns.md` 获取完整的通用规则集。

---

## 通用规则

每个生成 agent 或读取重要内容的工作流必须遵循以下规则：

1. **绝不**读取 agent 定义文件（`agents/*.md`）——`subagent_type` 会自动加载它们
2. **绝不**将大文件内联到子 agent 的 prompt 中——告诉 agent 从磁盘读取文件
3. **读取深度随上下文窗口伸缩**——检查 `.planning/config.json` 中的 `context_window`：
   - 在 < 500000 token（默认 200k）时：仅读取 frontmatter、状态字段或摘要。绝不读取完整的 SUMMARY.md、VERIFICATION.md 或 RESEARCH.md 正文。
   - 在 >= 500000 token（1M 模型）时：当内容需要内联呈现或用于决策时，可以读取完整的子 agent 输出正文。但仍避免不必要的读取。
4. **委托**繁重工作给子 agent——编排器负责路由，不执行
5. **主动警告**：如果你已经消耗了大量上下文（大文件读取、多个子 agent 结果），警告用户："上下文预算趋紧。建议检查点保存进度。"

## 按上下文窗口的读取深度

| 上下文窗口 | 子 agent 输出读取 | SUMMARY.md | VERIFICATION.md | PLAN.md（其他阶段） |
|---------------|------------------------|------------|-----------------|------------------------|
| < 500k（200k 模型） | 仅 Frontmatter | 仅 Frontmatter | 仅 Frontmatter | 仅当前阶段 |
| >= 500k（1M 模型） | 允许读取完整正文 | 允许读取完整正文 | 允许读取完整正文 | 仅当前阶段 |

**如何检查：** 读取 `.planning/config.json` 并检查 `context_window`。如果该字段缺失，按 200k 处理（保守默认值）。

## 上下文降级层级

监控上下文使用量并相应调整行为：

| 层级 | 使用量 | 行为 |
|------|-------|----------|
| PEAK | 0-30% | 全部操作。读取正文，生成多个 agent，内联结果。 |
| GOOD | 30-50% | 常规操作。优先读取 frontmatter，积极委托。 |
| DEGRADING | 50-70% | 节省。仅读取 frontmatter，最小内联，警告用户预算。 |
| POOR | 70%+ | 紧急模式。立即保存检查点进度。除非关键，否则不进行新的读取。 |

## 上下文降级预警信号

质量在触发告警阈值之前会逐渐退化。关注这些早期信号：

- **静默部分完成**——agent 声称任务完成但实现不完整。自检能捕获文件存在性但不能捕获语义完整性。始终验证 agent 输出是否符合计划的 must_haves，而不仅仅是文件是否存在。
- **越来越模糊**——agent 开始使用"适当的处理"或"标准模式"等措辞而非具体代码。这表明上下文压力，甚至早于预算警告触发。
- **跳过步骤**——agent 省略了通常会遵循的协议步骤。如果 agent 的成功标准有 8 项但只报告了 5 项，怀疑上下文压力。

委托给 agent 时，编排器无法验证 agent 输出的语义正确性——只能验证结构完整性。这是根本性限制。通过 must_haves.truths 和抽查验证来缓解。

## MCP 工具模式成本（运行框架关注点）

每个启用的 MCP 服务器都会将其工具模式注入到**每一轮对话**中，无论你是否调用其任何工具。重量级服务器每轮可达 20k+ token——通常远超 GSD 通过 `model_profile` 调优能节省的量。这是 Claude Code 运行框架的关注点，不是 GSD 的关注点：GSD**不**管理 MCP 启用。切换键位于 `.claude/settings.json` 中的 `enabledMcpjsonServers` 和 `disabledMcpjsonServers`。

### 为什么这是你无法控制的最大成本杠杆

工具模式会消耗与模型上下文、prompt 和对话历史相同的上下文预算。如果一个项目有 5 个未使用的 MCP 服务器，每个平均 5k token 的模式，则每轮都要支付 25k token 的税，甚至早于助手读取任何项目文件。精简 MCP 具有**乘数效应**，与你选择的 `model_profile` 组合叠加——无论使用哪个模型，每轮的开销都下降。

### 阶段前 MCP 审计

在开始一个长阶段之前（特别是 `/gsd-execute-phase`、`/gsd-plan-phase` 或任何跨越多个子 agent 的阶段），运行此审计：

- [ ] **浏览器/playwright 工具是否启用？** 如果此阶段没有 UI 工作，请禁用它们。它们是每轮模式最重的工具之一。
- [ ] **平台特定工具是否启用？** 当不需要为当前阶段主动使用时，应禁用 Mac-tools / Windows-tools / 操作系统特定的辅助工具。
- [ ] **跨项目/过时的 MCP？** 为其他项目添加的服务器仍然在此处启用。这些通常被遗忘，每轮都要支付税款却没有收益。
- [ ] **重复或阴影服务器？** 两个 MCP 提供相似的工具（例如两个不同的文件系统辅助工具）。保留一个。

每个被禁用的项目都会在后续会话的每一轮中移除其模式。

### 如何切换

键位于 `.claude/settings.json`（项目级别），**不**在 `.planning/config.json` 中：

```json
{
  "enabledMcpjsonServers": ["context7"],
  "disabledMcpjsonServers": ["playwright", "mac-tools"]
}
```

任选其一——`enabledMcpjsonServers` 是显式允许列表，`disabledMcpjsonServers` 是针对默认值的阻止列表。详见 [Claude Code MCP 文档](https://docs.anthropic.com/en/docs/claude-code/mcp)。

### 与 model_profile 的组合

精简 MCP 和调优 `model_profile` 是独立杠杆，**组合叠加**。禁用 25k token 的 MCP 每轮节省 25k，无论你运行的是 `quality`（到处 opus）还是 `budget`（sonnet/haiku）；节省是叠加的，不是替代模型调优。不要二选一——两者都做，并先审计 MCP，因为每轮的节省立竿见影，且叠加到编排器生成的每个子 agent 上。
