<purpose>
讨论阶段的超级用户模式。将所有问题预先生成为 JSON 状态文件和 HTML 伴侣 UI，然后等待用户按自己的节奏回答。当用户发出就绪信号时，一次性处理所有答案并生成 CONTEXT.md。

**适用场景：** 具有许多灰色区域的大型阶段，或用户更喜欢离线/异步回答问题而不是在聊天会话中交互式回答时。
</purpose>

<trigger>
当 ARGUMENTS 中存在 `--power` 标志时，此工作流执行。调用者（discuss-phase.md）已完成阶段验证和上下文提供。
</trigger>

<step name="analyze">
运行与标准讨论阶段模式相同的灰色区域识别。识别所有灰色区域并为每个生成 2-4 个具体选项及权衡描述。按主题分组为部分。
</step>

<step name="generate_json">
将所有问题写入 `{phase_dir}/{padded_phase}-QUESTIONS.json`，采用结构化 JSON 格式，包含阶段信息、统计数据、分节的问题（每个问题有 ID、标题、上下文、选项和状态字段）。
</step>

<step name="generate_html">
编写一个自包含的 HTML 伴侣文件到 `{phase_dir}/{padded_phase}-QUESTIONS.html`。包含内联 CSS 和 JavaScript，无外部依赖。布局包括统计栏（带进度条）、可折叠区域标题和问题卡片（包含选项单选按钮和备注文本区）。支持保存答案到 JSON 文件的机制。
</step>

<step name="notify_user">
通知用户已创建的文件和可用命令（refresh、finalize、explain Q-N、exit power mode）。
</step>

<step name="wait_loop">
进入等待模式，监听用户命令：
- "refresh" — 重新读取 JSON，更新统计和 HTML
- "finalize" — 继续执行 finalize 步骤以生成 CONTEXT.md
- "explain Q-{N}" — 提供特定问题的详细解释
- "exit power mode" — 切换到标准交互式讨论
</step>

<step name="finalize">
处理所有已回答的问题并生成 CONTEXT.md。按部分分组决策，格式化决策条目，写入标准 context 模板格式。如果回答的问题少于 50%，发出警告。
</step>

<success_criteria>
- [ ] Questions generated into well-structured JSON covering all identified gray areas
- [ ] HTML companion file is self-contained and usable without a server
- [ ] Stats bar accurately reflects answered/remaining counts after each refresh
- [ ] Answered questions highlighted green in HTML
- [ ] CONTEXT.md generated in the same format as standard discuss-phase output
- [ ] Unanswered questions preserved as deferred items (not silently dropped)
- [ ] canonical_refs section always present in CONTEXT.md (MANDATORY)
- [ ] User knows how to refresh, finalize, explain, or exit power mode
</success_criteria>
