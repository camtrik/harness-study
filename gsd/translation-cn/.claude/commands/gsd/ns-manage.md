---
name: gsd-manage
description: "配置 工作区 | 工作流 thread update ship inbox"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

根据用户意图路由到相应的管理 skill。
`gsd-config`（settings + advanced + integrations + profile）和 `gsd-workspace`
（new + list + remove）是 #2790 之后合并的入口。

| 用户想要 | 调用 |
|---|---|
| 配置 GSD 设置（基础/高级/集成/配置文件） | gsd-config |
| 管理工作区（创建/列出/删除） | gsd-workspace |
| 管理并行工作流 | gsd-workstreams |
| 在新的上下文线程中继续工作 | gsd-thread |
| 暂停当前工作 | gsd-pause-work |
| 恢复暂停的工作 | gsd-resume-work |
| 更新 GSD 安装 | gsd-update |
| 交付已完成的工作 | gsd-ship |
| 处理收件箱项目 | gsd-inbox |
| 创建干净的 PR 分支 | gsd-pr-branch |
| 撤销上次 GSD 操作 | gsd-undo |

使用 Skill 工具直接调用匹配的 skill。
