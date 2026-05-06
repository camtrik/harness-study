<purpose>
苏格拉底式创意工作流。通过探究性问题引导开发者探索一个想法，在有用时提供对话中的研究，然后将结晶的输出路由到 GSD 产物。
</purpose>

<required_reading>
阅读调用 prompt 的 execution_context 中引用的所有文件。

@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/questioning.md
@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/domain-probes.md
</required_reading>

<available_agent_types>
- gsd-phase-researcher — 研究特定问题并返回简洁的发现
</available_agent_types>

<process>

## 步骤 1：开启对话

如果提供了主题，确认并开始探索。如果无主题，询问开发者在想什么。

## 步骤 2：苏格拉底式对话（2-5 轮交流）

使用 `questioning.md` 和 `domain-probes.md` 的原则引导对话：一次问一个问题，探测约束、权衡、用户、范围、依赖、风险。对话应感觉自然而非公式化。

## 步骤 3：对话中的研究提议（2-3 轮交流后）

如果对话浮现了事实性问题或技术比较，提议快速研究（约 30 秒）。如果同意，启动 gsd-phase-researcher subagent。分享发现并继续对话。

## 步骤 4：结晶输出（3-6 轮交流后）

当对话达到自然结论时，分析讨论了什么并提出最多 4 个输出：笔记、待办、种子、研究问题、需求、新阶段、Spike 或 Sketch。向用户呈现建议。

## 步骤 5：写入选定的输出

为用户选择的每个输出写入文件到正确目的地。

## 步骤 6：关闭

呈现探索完成摘要。
</process>

<success_criteria>
- [ ] 苏格拉底式对话遵循 questioning.md 原则
- [ ] 问题一次一个，非批量
- [ ] 研究在上下文中提议（非强制）
- [ ] 从对话中提出最多 4 个输出
- [ ] 用户明确选择要创建哪些输出
- [ ] 文件写入正确目的地
- [ ] 提交遵循 commit_docs 配置
</success_criteria>
