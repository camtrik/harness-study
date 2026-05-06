# GSD 指令

- 当用户请求 GSD 或使用 `gsd-*` 命令时，使用 get-shit-done skill。
- 将 `/gsd-...` 或 `gsd-...` 视为命令调用，并从 `.github/skills/gsd-*` 加载匹配的文件。
- 当命令指示生成 subagent 时，优先选择 `.github/agents` 中的匹配自定义 agent。
- 除非用户明确要求，否则不要应用 GSD 工作流。
- 完成任何 `gsd-*` 命令（或其触发的任何交付物：功能、bug 修复、测试、文档等）后，始终：(1) 通过 `ask_user` 提示，向用户提供下一步建议；重复此反馈循环，直到用户明确表示完成。
