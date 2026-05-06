# 工作流标志（`--ws`）

## 概览

`--ws <name>` 标志将 GSD 操作限定到特定工作流（workstream），使多个 Claude Code 实例能在同一代码库上并行进行里程碑工作。

## 解析优先级

1. `--ws <name>` 标志（显式，最高优先级）
2. `GSD_WORKSTREAM` 环境变量（按实例）
3. 临时存储中的会话范围活跃工作流指针（按运行时会话 / 终端）
4. `.planning/active-workstream` 文件（当没有会话键存在时的旧版共享回退）
5. `null`——扁平模式（无工作流）

## 为什么存在会话范围指针

共享的 `.planning/active-workstream` 文件在多个 Claude/Codex 实例在同一仓库上同时活跃时从根本上不安全。一个会话可以静默地重新指向另一个会话的 `STATE.md`、`ROADMAP.md` 和阶段路径。

GSD 现在优先使用由运行时/会话标识键控的会话范围指针（`GSD_SESSION_KEY`、`CODEX_THREAD_ID`、`CLAUDE_CODE_SSE_PORT`、终端会话 ID 或控制 TTY）。这在保持并发会话隔离的同时，为不暴露稳定会话键的运行时保留了旧版兼容性。

## 会话标识解析

当 GSD 在上述步骤 3 中解析会话范围指针时，使用以下顺序：

1. 显式运行时/会话环境变量如 `GSD_SESSION_KEY`、`CODEX_THREAD_ID`、`CLAUDE_SESSION_ID`、`CLAUDE_CODE_SSE_PORT`、`OPENCODE_SESSION_ID`、`GEMINI_SESSION_ID`、`CURSOR_SESSION_ID`、`WINDSURF_SESSION_ID`、`TERM_SESSION_ID`、`WT_SESSION`、`TMUX_PANE` 和 `ZELLIJ_SESSION_NAME`
2. 如果 shell/运行时已暴露终端路径则使用 `TTY` 或 `SSH_TTY`
3. 仅在 stdin 是交互式时进行单次尽力而为的 `tty` 探测

如果以上都未产生稳定标识，GSD 不会继续探测。它直接回退到旧版共享 `.planning/active-workstream` 文件。

## 指针生命周期

会话范围指针是有意轻量和尽力而为的：

- 为一个会话清除工作流仅移除该会话的指针文件
- 如果那是该仓库的最后一个指针，GSD 也会移除现在为空的按项目临时目录
- 如果仍有兄弟会话指针存在，临时目录保留
- 当指针引用不再存在的工作流目录时，GSD 将其视为陈旧状态：移除该指针文件并解析为 `null`，直到会话再次显式设置新的活跃工作流

## 路由传播

所有工作流路由命令包含 `${GSD_WS}`，它：
- 当工作流活跃时展开为 `--ws <name>`
- 在扁平模式下展开为空字符串（向后兼容）

## 目录结构

```
.planning/
├── PROJECT.md          # 共享
├── config.json         # 共享
├── milestones/         # 共享
├── codebase/           # 共享
├── active-workstream   # 仅旧版共享回退
└── workstreams/
    ├── feature-a/      # 工作流 A
    │   ├── STATE.md
    │   ├── ROADMAP.md
    │   ├── REQUIREMENTS.md
    │   └── phases/
    └── feature-b/      # 工作流 B
        ├── STATE.md
        ├── ROADMAP.md
        ├── REQUIREMENTS.md
        └── phases/
```

## CLI 用法

```bash
# 所有 gsd-sdk query 命令接受 --ws
gsd-sdk query state.json --ws feature-a
gsd-sdk query find-phase 3 --ws feature-b

# 会话本地切换，无需每次命令都加 --ws
GSD_SESSION_KEY=my-terminal-a gsd-sdk query workstream.set feature-a
GSD_SESSION_KEY=my-terminal-a gsd-sdk query state.json
GSD_SESSION_KEY=my-terminal-b gsd-sdk query workstream.set feature-b
GSD_SESSION_KEY=my-terminal-b gsd-sdk query state.json

# 工作流 CRUD
gsd-sdk query workstream.create <name>
gsd-sdk query workstream.list
gsd-sdk query workstream.status <name>
gsd-sdk query workstream.complete <name>
```
