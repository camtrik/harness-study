# 模型配置文件解析

在编排开始时解析模型配置一次，然后用于所有 Task 生成。

## 解析模式

```bash
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
```

默认值：如果未设置或配置缺失，使用 `balanced`。

## 查找表

@/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/references/model-profiles.md

在表中查找已解析配置文件中 agent 对应的模型。将模型参数传递给 Task 调用：

```
Task(
  prompt="...",
  subagent_type="gsd-planner",
  model="{resolved_model}"  # "inherit"、"sonnet" 或 "haiku"
)
```

**注意：** Opus 级别的 agent 解析为 `"inherit"`（而非 `"opus"`）。这使 agent 使用父会话的模型，避免可能与阻止特定 opus 版本的组织策略冲突。

如果 `model_profile` 是 `"adaptive"`，agent 按基于角色的分配解析（根据 agent 类型分配 opus/sonnet/haiku）。

如果 `model_profile` 是 `"inherit"`，所有 agent 解析为 `"inherit"`（对 OpenCode `/model` 有用）。

## 用法

1. 在编排开始时解析一次
2. 存储配置值
3. 生成时在表中查找每个 agent 的模型
4. 将模型参数传递给每个 Task 调用（值：`"inherit"`、`"sonnet"`、`"haiku"`）
