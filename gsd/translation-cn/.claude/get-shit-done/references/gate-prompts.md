# 门提示模式

工作流和 agent 中结构化门检查的可复用提示模式。

**关于检查点框格式详情，参见 `references/ui-brand.md`**——检查点框使用双线边框绘制，内部宽度为 62 字符。

## 规则

- `header` 必须最多 12 个字符
- 门检查中 `multiSelect` 始终为 `false`
- 始终处理 "Other" 情况（用户输入了自由形式响应而非选择）
- 每个提示最多 4 个选项——如果需要更多，使用两步流程

---

## 模式：approve-revise-abort
3 选项门，用于计划审批、间隙关闭审批。
- question："批准这些 {名词}？"
- header："批准？"
- options：批准 | 请求修改 | 中止

## 模式：yes-no
简单的 2 选项确认，用于重新规划、重建、替换计划、提交。
- question："{关于操作的具体问题}"
- header："确认"
- options：是 | 否

## 模式：stale-continue
2 选项刷新门，用于陈旧警告、时间戳新鲜度。
- question："{产物} 可能已过时。刷新还是继续？"
- header："陈旧"
- options：刷新 | 继续使用

## 模式：yes-no-pick
3 选项选择，用于种子选择、项目包含。
- question："在规划中包含 {项目}？"
- header："包含？"
- options：是，全部 | 让我选 | 否

## 模式：multi-option-failure
4 选项失败处理器，用于构建失败。
- question："计划 {id} 失败。我们该如何继续？"
- header："失败"
- options：重试 | 跳过 | 回滚 | 中止

## 模式：multi-option-escalation
4 选项升级，用于审查升级（超过最大重试次数）。
- question："阶段 {N} 已失败验证 {attempt} 次。我们该如何继续？"
- header："升级"
- options：接受差距 | 重新规划（通过 /gsd-plan-phase） | 调试（通过 /gsd-debug） | 重试

## 模式：multi-option-gaps
4 选项差距处理器，用于审查 gaps-found。
- question："{count} 个验证差距需要处理。我们该如何继续？"
- header："差距"
- options：自动修复 | 覆盖 | 手动 | 跳过

## 模式：multi-option-priority
4 选项优先级选择，用于里程碑差距优先级。
- question："我们应该解决哪些差距？"
- header："优先级"
- options：仅必须修复 | 必须 + 应该 | 全部 | 让我选

## 模式：toggle-confirm
2 选项确认，用于启用/禁用布尔功能。
- question："启用 {功能名称}？"
- header："切换"
- options：启用 | 禁用

## 模式：action-routing
最多 4 个建议的下步操作及选择（状态、恢复工作流）。
- question："接下来你想做什么？"
- header："下一步"
- options：{主要操作} | {替代 1} | {替代 2} | 其他
- 注意：从工作流状态动态生成选项。始终将"其他"作为最后一个选项。

## 模式：scope-confirm
3 选项确认，用于快速任务范围验证。
- question："这个任务看起来很复杂。作为快速任务进行还是使用完整规划？"
- header："范围"
- options：快速任务 | 完整计划（通过 /gsd-plan-phase） | 修改

## 模式：depth-select
3 选项深度选择，用于规划工作流偏好。
- question："规划应该有多彻底？"
- header："深度"
- options：快速（3-5 个阶段，跳过研究） | 标准（5-8 个阶段，默认） | 全面（8-12 个阶段，深入研究）

## 模式：context-handling
3 选项处理器，用于讨论工作流中已有的 CONTEXT.md。
- question："阶段 {N} 已有 CONTEXT.md。应如何处理？"
- header："上下文"
- options：覆盖 | 追加 | 取消

## 模式：gray-area-option
动态模板，用于在讨论工作流中呈现灰色区域选择。
- question："{灰色区域标题}"
- header："决策"
- options：{选项 1} | {选项 2} | 让 Claude 决定
- 注意：选项在运行时生成。始终将"让 Claude 决定"作为最后一个选项。
