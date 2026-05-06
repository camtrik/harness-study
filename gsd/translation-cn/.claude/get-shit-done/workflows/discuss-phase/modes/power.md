# --power 模式 — 批量问题生成，异步回答

> **延迟加载。** 当 `$ARGUMENTS` 中存在 `--power` 时，从 `workflows/discuss-phase.md` 读取此文件。完整的分步说明位于现有的 `discuss-phase-power.md` 工作流文件中（保持在其原始路径，以便已安装的 `@` 引用继续解析）。

## 分派

```
Read @/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/workflows/discuss-phase-power.md
```

端到端执行它。不要继续标准交互式步骤。

## 流程摘要

超级用户模式将所有问题预先生成为机器可读和人类友好的文件，然后等待用户按自己的节奏回答，最后一次性处理所有答案。

1. 运行与标准模式相同的阶段分析（灰色区域识别）
2. 将所有问题写入 `{phase_dir}/{padded_phase}-QUESTIONS.json` 和 `{phase_dir}/{padded_phase}-QUESTIONS.html`
3. 通知用户文件路径并等待"refresh"或"finalize"命令
4. 收到"refresh"：读取 JSON，处理已回答的问题，更新统计和 HTML
5. 收到"finalize"：从 JSON 读取所有答案，以标准格式生成 CONTEXT.md

## 适用场景

具有许多灰色区域的大型阶段，或用户更喜欢离线/异步回答问题而不是在聊天会话中交互式回答。

## 组合规则

- `--power --auto`：power 胜出。Power 模式与自主选择不兼容 — 其目的是离线回答。
- `--power --chain`：在 power 模式 finalize 步骤写入 CONTEXT.md 后，链自动推进仍然适用。
