# 步骤：codebase_drift_gate

执行后结构漂移检测（#2003）。在最后一批提交完成后、验证之前运行。**按合同不具有阻塞性：** 此处的任何内部错误必须静默继续到 `verify_phase_goal`。阶段永远不会因为此门而失败。

```bash
DRIFT=$(gsd-sdk query verify.codebase-drift 2>/dev/null || echo '{"skipped":true,"reason":"sdk-failed"}')
```

解析 JSON 获取各种字段（skipped、reason、action_required、directive 等）。

**如果 `skipped` 为 true：** 记录一行日志并继续到 `verify_phase_goal`。不提示用户。不阻塞。

**如果 `action_required` 为 false：** 静默继续。

**如果 `action_required` 为 true 且 `directive` 为 `warn`：** 原样打印 message 字段。不阻塞。不启动任何东西。

**如果 `action_required` 为 true 且 `directive` 为 `auto-remap`：** 启动 gsd-codebase-mapper agent 进行增量重新映射。如果启动失败，记录并继续。重新映射成功则记录并继续。

此步骤是完全非阻塞的 — 它永远不会导致阶段失败，任何异常路径都将控制权返回给 `verify_phase_goal`。

相关配置键：
- `workflow.drift_threshold`（整数，默认 3）— 触发操作所需的最小漂移元素数
- `workflow.drift_action` — `warn`（默认）或 `auto-remap`
