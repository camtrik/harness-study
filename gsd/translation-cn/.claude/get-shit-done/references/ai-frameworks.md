# AI 框架决策矩阵

> 供 `gsd-framework-selector` 和 `gsd-ai-researcher` 使用的参考文档。
> 提炼自官方文档、基准测试和开发者报告（2026 年）。

---

## 快速选择

| 情况 | 选择 |
|-----------|------|
| 构建可用 agent 的最简路径（OpenAI） | OpenAI Agents SDK |
| 构建可用 agent 的最简路径（模型无关） | CrewAI |
| 生产环境 RAG / 文档问答 | LlamaIndex |
| 复杂有状态工作流（含分支） | LangGraph |
| 具有明确角色的多 agent 团队 | CrewAI |
| 代码感知的自主 agent（Anthropic） | Claude Agent SDK |
| "我还不知道需求" | LangChain |
| 受监管 / 需要审计追踪 | LangGraph |
| 企业 Microsoft/.NET 团队 | AutoGen/AG2 |
| Google Cloud / Gemini 承诺用户 | Google ADK |
| 需要显式控制的纯 NLP 流水线 | Haystack |

---

## 框架概览

### CrewAI
- **类型：** 多 agent 编排
- **语言：** 仅 Python
- **模型支持：** 模型无关
- **学习曲线：** 初级（角色/任务/crew 映射到真实团队）
- **最适合：** 内容流水线、研究自动化、业务流程工作流、快速原型
- **不适合：** 细粒度状态管理、TypeScript、容错检查点、复杂条件分支
- **优势：** 最快的多 agent 原型速度，在 QA 任务上比 LangGraph 快 5.76 倍，内置记忆（短期/长期/实体/上下文）、Flows 架构、独立（不依赖 LangChain）
- **劣势：** 检查点能力有限，错误处理粗糙，仅支持 Python
- **评估关注点：** 任务分解准确性、agent 间交接、目标完成率、循环检测

### LlamaIndex
- **类型：** RAG 和数据摄取
- **语言：** Python + TypeScript
- **模型支持：** 模型无关
- **学习曲线：** 中级
- **最适合：** 法律研究、内部知识助手、企业文档搜索、任何以检索质量为第一优先的系统
- **不适合：** 主要需求是 agent 编排、多 agent 协作或聊天机器人对话流
- **优势：** 一流文档解析（LlamaParse）、检索准确率提升 35%、查询速度提升 20-30%、混合检索策略（向量 + 图 + 重排序）
- **劣势：** 数据框架优先——agent 编排是次要的
- **评估关注点：** 上下文忠实度、幻觉、答案相关性、检索精确度/召回率

### LangChain
- **类型：** 通用 LLM 框架
- **语言：** Python + TypeScript
- **模型支持：** 模型无关（最广泛的生态系统）
- **学习曲线：** 中级至高级
- **最适合：** 需求演变中、需要大量第三方集成、希望用一个框架满足所有需求的团队、RAG + agent + chain
- **不适合：** 简单的明确用例、RAG 为主（用 LlamaIndex）、复杂有状态工作流（用 LangGraph）、大规模性能关键
- **优势：** 最大的社区和集成生态系统、比从零开始快 25%、覆盖 RAG/agent/chain/记忆
- **劣势：** 抽象层开销、负载下 p99 延迟恶化、复杂度蔓延风险
- **评估关注点：** 端到端任务完成度、chain 正确性、检索质量

### LangGraph
- **类型：** 有状态 agent 工作流（基于图）
- **语言：** Python + TypeScript（完全对等）
- **模型支持：** 模型无关（继承 LangChain 集成）
- **学习曲线：** 中级至高级（图的思维模型）
- **最适合：** 生产级有状态工作流、受监管行业、审计追踪、人机协同流程、容错多步骤 agent
- **不适合：** 简单聊天机器人、纯线性工作流、快速原型
- **优势：** 最好的检查点机制（每个节点）、时间旅行调试、原生 Postgres/Redis 持久化、流式支持、62% 的开发者选用于有状态 agent 工作（2026 年）
- **劣势：** 前期搭建工作更多、学习曲线更陡、对简单场景过度设计
- **评估关注点：** 状态转换正确性、目标完成率、工具使用准确性、安全护栏

### OpenAI Agents SDK
- **类型：** 原生 OpenAI agent 框架
- **语言：** Python + TypeScript
- **模型支持：** 针对 OpenAI 优化（通过 Chat Completions 兼容性支持 100+ 模型）
- **学习曲线：** 初级（4 个原语：Agents、Handoffs、Guardrails、Tracing）
- **最适合：** 承诺使用 OpenAI 的团队、快速 agent 原型、语音 agent（gpt-realtime）、希望使用可视化构建器（AgentKit）的团队
- **不适合：** 需要模型灵活性、复杂多 agent 协作、需要持久状态管理、担心供应商锁定
- **优势：** 最简单的思维模型、内置追踪和护栏、Handoffs 用于 agent 委派、Realtime Agents 用于语音
- **劣势：** OpenAI 供应商锁定、无内置持久状态、生态系统较年轻
- **评估关注点：** 指令遵循、安全护栏、升级准确性、语气一致性

### Claude Agent SDK（Anthropic）
- **类型：** 代码感知的自主 agent 框架
- **语言：** Python + TypeScript
- **模型支持：** 仅 Claude 模型
- **学习曲线：** 中级（18 个 hook 事件、MCP、工具装饰器）
- **最适合：** 开发者工具、代码生成/审查 agent、自主编码助手、MCP 密集型架构、安全关键应用
- **不适合：** 需要模型灵活性、需要稳定/成熟的 API、与代码/工具使用无关的用例
- **优势：** 最深度的 MCP 集成、内置文件系统/shell 访问、18 个生命周期 hook、自动上下文压缩、扩展思考、安全优先设计
- **劣势：** 仅 Claude 供应商锁定、API 较新/演变中、社区较小
- **评估关注点：** 工具使用正确性、安全性、代码质量、指令遵循

### AutoGen / AG2 / Microsoft Agent Framework
- **类型：** 多 agent 对话框架
- **语言：** Python（AG2）、Python + .NET（Microsoft Agent Framework）
- **模型支持：** 模型无关
- **学习曲线：** 中级至高级
- **最适合：** 研究应用、对话式问题解决、代码生成 + 执行循环、Microsoft/.NET 团队
- **不适合：** 追求生态系统稳定性、确定性工作流或"最安全的长期选择"（存在分裂风险）
- **优势：** 最复杂的对话式 agent 模式、代码生成 + 执行循环、异步事件驱动（v0.4+）、跨语言互操作（Microsoft Agent Framework）
- **劣势：** 生态系统分裂（AutoGen 维护模式、AG2 分支、Microsoft Agent Framework 预览）——真实的长期风险
- **评估关注点：** 对话目标完成度、共识质量、代码执行正确性

### Google ADK（Agent Development Kit）
- **类型：** 多 agent 编排框架
- **语言：** Python + Java
- **模型支持：** 针对 Gemini 优化；通过 LiteLLM 支持其他模型
- **学习曲线：** 中级（agent/工具/会话模型，如果了解 LangGraph 则容易上手）
- **最适合：** Google Cloud / Vertex AI 团队、需要内置会话管理和记忆的多 agent 工作流、已承诺使用 Gemini 的团队、需要 Google Search / BigQuery 工具集成的 agent 流水线
- **不适合：** 需要 Gemini 之外的模型灵活性、不能接受 Google Cloud 依赖、纯 TypeScript 技术栈
- **优势：** Google 官方支持、内置会话/记忆/产物管理、紧密的 Vertex AI 和 Google Search 集成、自有评估框架（兼容 RAGAS）、原生多 agent 设计（顺序、并行、循环模式）、Java SDK 适用于企业团队
- **劣势：** 实践中 Gemini 供应商锁定、社区比 LangChain/LlamaIndex 年轻、第三方集成深度不足
- **评估关注点：** 多 agent 任务分解、工具使用正确性、会话状态一致性、目标完成率

### Haystack
- **类型：** NLP 流水线框架
- **语言：** Python
- **模型支持：** 模型无关
- **学习曲线：** 中级
- **最适合：** 需要显式、可审计的 NLP 流水线、需要细粒度控制的文档处理、企业搜索、需要透明度的受监管行业
- **不适合：** 快速原型、多 agent 工作流、或希望拥有大型社区
- **优势：** 显式的流水线控制、结构化数据流水线能力强、文档优秀
- **劣势：** 社区较小、agent 能力不如其他替代方案
- **评估关注点：** 提取准确率、流水线输出有效性、检索质量

---

## 决策维度

### 按系统类型

| 系统类型 | 主要框架 | 关键评估关注点 |
|-------------|---------------------|-------------------|
| RAG / 知识问答 | LlamaIndex、LangChain | 上下文忠实度、幻觉、检索精确度/召回率 |
| 多 agent 编排 | CrewAI、LangGraph、Google ADK | 任务分解、交接质量、目标完成度 |
| 对话助手 | OpenAI Agents SDK、Claude Agent SDK | 语气、安全性、指令遵循、升级判断 |
| 结构化数据提取 | LangChain、LlamaIndex | 模式合规、提取准确率 |
| 自主任务 agent | LangGraph、OpenAI Agents SDK | 安全护栏、工具正确性、成本控制 |
| 内容生成 | Claude Agent SDK、OpenAI Agents SDK | 品牌调性、事实准确性、语气 |
| 代码自动化 | Claude Agent SDK | 代码正确性、安全性、测试通过率 |

### 按团队规模和阶段

| 上下文 | 推荐 |
|---------|----------------|
| 个人开发者，原型阶段 | OpenAI Agents SDK 或 CrewAI（最快上手） |
| 个人开发者，RAG | LlamaIndex（内置功能丰富） |
| 团队，生产环境，有状态 | LangGraph（最好的容错能力） |
| 团队，需求演变中 | LangChain（最广泛的逃生通道） |
| 团队，多 agent | CrewAI（最简单的角色抽象） |
| 企业，.NET | AutoGen AG2 / Microsoft Agent Framework |

### 按模型承诺

| 偏好 | 框架 |
|-----------|-----------|
| 仅 OpenAI | OpenAI Agents SDK |
| 仅 Anthropic/Claude | Claude Agent SDK |
| Google/Gemini 承诺用户 | Google ADK |
| 模型无关（完全灵活） | LangChain、LlamaIndex、CrewAI、LangGraph、Haystack |

---

## 反模式

1. **用 LangChain 做简单聊天机器人**——直接 SDK 调用代码更少、更快、更易调试
2. **用 CrewAI 做复杂有状态工作流**——检查点能力不足会在生产环境中出问题
3. **用 OpenAI Agents SDK 对接非 OpenAI 模型**——失去了你选择它的集成优势
4. **将 LlamaIndex 作为多 agent 框架**——它可以做 agent，但这不是它的强项
5. **不评估替代方案就默认使用 LangChain**——"大家都在用"不等于适合你的用例
6. **在新项目中使用 AutoGen（而非 AG2）**——AutoGen 处于维护模式；使用 AG2 或等待 Microsoft Agent Framework 正式发布
7. **用 LangGraph 做简单线性流程**——图的额外开销不值得；使用 LangChain chain 即可
8. **忽视供应商锁定**——供应商原生 SDK（OpenAI、Claude）以灵活性换取集成深度；要有意识地做出选择

---

## 组合策略（多框架技术栈）

| 生产模式 | 技术栈 |
|-------------------|-------|
| RAG + 可观测性 | LlamaIndex + LangSmith 或 Langfuse |
| 有状态 agent + RAG | LangGraph + LlamaIndex |
| 多 agent + 追踪 | CrewAI + Langfuse |
| OpenAI agent + 评估 | OpenAI Agents SDK + Promptfoo 或 Braintrust |
| Claude agent + MCP | Claude Agent SDK + LangSmith 或 Arize Phoenix |
