# 代码库扫视——映射选择表

> `workflows/discuss-phase.md` 中 `scout_codebase` 步骤的延迟加载参考
> （通过 #2551 渐进披露重构提取）。仅当先前的 `.planning/codebase/*.md` 映射存在
> 且工作流需要选择加载哪 2-3 个时读取。

## 阶段类型→推荐映射

根据推断的阶段类型读取 2-3 个映射。不要读取全部七个——
那会膨胀上下文而不提高讨论质量。

| 阶段类型（从标题 + ROADMAP 条目推断） | 读取这些映射 |
|---|---|
| UI / 前端 / 样式 / 设计 | CONVENTIONS.md、STRUCTURE.md、STACK.md |
| 后端 / API / 服务 / 数据模型 | STACK.md、ARCHITECTURE.md、INTEGRATIONS.md |
| 集成 / 第三方 / 供应商 | STACK.md、INTEGRATIONS.md、ARCHITECTURE.md |
| 基础设施 / DevOps / CI / 部署 | STACK.md、ARCHITECTURE.md、INTEGRATIONS.md |
| 测试 / QA / 覆盖率 | TESTING.md、CONVENTIONS.md、STRUCTURE.md |
| 文档 / 内容 | CONVENTIONS.md、STRUCTURE.md |
| 混合 / 不清楚 | STACK.md、ARCHITECTURE.md、CONVENTIONS.md |

仅在阶段明确处理已知关注点或安全问题时才读取 CONCERNS.md。

## 单次读取规则

在**单次** Read 调用中读取每个映射文件。不要以两个不同的偏移量读取同一文件——
分段读取破坏 prompt 缓存复用且成本高于单次完整读取。

## 无映射的回退

如果 `.planning/codebase/*.md` 不存在：
1. 从阶段目标中提取关键术语（例如 "feed" → "post"、"card"、"list"；"auth" → "login"、"session"、"token"）
2. `grep -rlE "{term1}|{term2}" src/ app/ --include="*.ts" ...`（使用 `-E` 进行扩展正则，使 `|` 交替在 GNU grep 和 BSD grep / macOS 上均适用），并 `ls` 常规的 component/hook/util 目录
3. 读取 3-5 个最相关的文件

## 输出（内部 `<codebase_context>`）

从扫描中识别：
- **可复用资产**——在此阶段可用的组件、hook、工具
- **既定模式**——状态管理、样式处理、数据获取
- **集成点**——新代码连接的路由、导航、提供者
- **创造性选项**——架构启用或约束的方法

在 `analyze_phase` 和 `present_gray_areas` 中使用。不写入文件——仅限会话内。
