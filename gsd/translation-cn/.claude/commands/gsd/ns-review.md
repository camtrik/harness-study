---
name: gsd-review
description: "质量关卡 | 代码审查 debug 审计 安全 eval ui"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的质量/审查 skill。
`gsd-code-review-fix` 在 #2790 中被 `gsd-code-review --fix` 吸收。

| 用户想要 | 调用 |
|---|---|
| 审查代码质量和正确性 | gsd-code-review |
| 自动修复代码审查发现 | gsd-code-review --fix |
| 审计 UAT / 验收测试 | gsd-audit-uat |
| 阶段安全审查 | gsd-secure-phase |
| 评估 AI 响应质量 | gsd-eval-review |
| 审查 UI 设计和可访问性 | gsd-ui-review |
| 验证阶段输出 | gsd-validate-phase |
| 调试失败的功能或错误 | gsd-debug |
| 对损坏系统进行取证调查 | gsd-forensics |

使用 Skill 工具直接调用匹配的 skill。
