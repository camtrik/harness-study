# 领域感知探询模式

`/gsd-begin`、`/gsd-discuss-phase` 和领域探索工作流的共享参考。

当用户提到某个技术领域时，使用这些探询提出有洞察力的后续问题。不要逐个对照清单——根据上下文选择最相关的 2-3 个。目标是揭示用户可能尚未考虑的隐藏假设和权衡。

---

## 认证

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "login" 或 "auth" | OAuth（哪些提供商？）、JWT，还是基于会话？需要社交登录还是仅邮箱/密码？ |
| "users" 或 "accounts" | 需要多因素认证吗？密码重置流程？邮箱验证？ |
| "sessions" | 会话持续时间和刷新策略？服务端会话还是无状态 token？ |
| "roles" 或 "permissions" | RBAC、ABAC，还是简单角色检查？有多少种不同角色？ |
| "API keys" | Key 轮换策略？每个 key 的作用域权限？每个 key 的速率限制？ |

---

## 实时更新

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "real-time" 或 "live updates" | WebSocket、SSE，还是轮询？具体什么需要实时 vs 最终一致？ |
| "notifications" | 推送通知（浏览器/移动端）、仅应用内，还是两者都要？持久化和已读回执？ |
| "collaboration" 或 "multiplayer" | 冲突解决策略？操作转换还是 CRDT？预期同时在线用户数？ |
| "chat" 或 "messaging" | 消息历史和搜索？输入中指示器？已读回执？ |
| "streaming" | 重连策略？连接断开时怎么办——排队还是丢弃？ |

---

## 仪表盘

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "dashboard" | 那些数据源输入？有多少个不同的视图？ |
| "charts" 或 "graphs" | 交互式还是静态？下钻能力？导出 CSV/PDF？ |
| "metrics" 或 "KPIs" | 刷新策略——实时、定期轮询、还是按需？可接受的延迟？ |
| "admin panel" | 基于角色的可见性？除了查看之外有哪些操作（编辑、删除、审批）？ |
| "mobile" 或 "responsive" | 简化的移动视图还是完全对等？图表的触控交互？ |

---

## API 设计

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "API" | REST、GraphQL，还是 RPC 风格？仅内部使用还是对外公开？ |
| "endpoints" 或 "routes" | 版本策略（URL 路径、Header、查询参数）？破坏性变更策略？ |
| "pagination" | 基于游标还是偏移？预期结果集大小？稳定的排序保证？ |
| "rate limiting" | 按用户、按 IP、还是按 API key？突发容忍度？如何向客户端传达限制？ |
| "errors" | 结构化错误格式？错误码 vs 错误消息？生产环境错误中暴露多少细节？ |

---

## 数据库

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "database" 或 "storage" | SQL 还是 NoSQL？选择的驱动因素——关系完整性、灵活性、规模？ |
| "ORM" 或 "queries" | ORM（哪一个？）还是原生查询？查询构建器作为中间方案？ |
| "migrations" | 迁移工具？回滚策略？如何处理数据迁移与模式迁移？ |
| "seeding" 或 "test data" | 开发环境的种子数据？逼真的假数据还是最小测试数据？ |
| "scale" 或 "performance" | 读写比？只读副本？连接池策略？ |

---

## 搜索

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "search" | 全文还是精确匹配？专用搜索引擎（Elasticsearch、Meilisearch）还是数据库层面？ |
| "filtering" 或 "facets" | 分面过滤？多少个过滤维度？组合过滤（AND/OR）？ |
| "autocomplete" 或 "typeahead" | 防抖策略？最小字符阈值？结果排序？ |
| "indexing" | 索引大小和更新频率？实时索引还是批量？可接受的索引延迟？ |
| "fuzzy" 或 "typo tolerance" | 模糊匹配？同义词支持？语言特定的词干？ |

---

## 文件上传/存储

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "upload" 或 "file upload" | 本地文件系统还是云存储（S3、GCS、Azure Blob）？直接上传还是通过服务器？ |
| "images" 或 "media" | 处理流水线——调整大小、压缩、生成缩略图？格式转换？ |
| "size limits" | 最大文件大小？每用户最大总存储？达到限制会发生什么？ |
| "CDN" | CDN 分发？更新文件的缓存失效？签名 URL 用于访问控制？ |
| "documents" 或 "attachments" | 病毒扫描？预览生成？上传文件的版本控制？ |

---

## 缓存

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "caching" 或 "performance" | 在哪里缓存——浏览器、CDN、应用层、数据库查询缓存？ |
| "invalidation" | 失效策略——TTL、事件驱动、还是手动？旁路缓存 vs 直写？ |
| "stale data" | 可接受的延迟窗口？延迟且重新验证模式？ |
| "Redis" 或 "Memcached" | 缓存拓扑——单节点还是集群？需要持久化还是纯缓存？ |
| "CDN" 或 "edge" | 静态资源的边缘缓存？边缘的动态内容？缓存 key 策略？ |

---

## 测试

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "testing" 或 "tests" | 单元、集成和 E2E 的平衡？你在哪里投入最多的测试精力？ |
| "mocking" 或 "stubs" | Mock 外部服务还是使用测试容器？数据库 mock 策略？ |
| "CI" 或 "pipeline" | CI 中运行测试？并行测试执行？按 PR 测试还是按推送测试？ |
| "coverage" | 覆盖率目标？覆盖率作为门控还是咨询？哪些指标（行、分支、函数）？ |
| "E2E" 或 "browser testing" | Playwright、Cypress 还是其他？有头还是无头？视觉回归测试？ |

---

## 部署

| 用户提及 | Agent 用领域知识探询 |
|---|---|
| "deploy" 或 "hosting" | 容器、Serverless、还是传统 VM/VPS？托管平台（Vercel、Railway）还是自托管？ |
| "CI/CD" 或 "pipeline" | GitHub Actions、GitLab CI 还是其他？合并到 main 时部署还是手动触发？ |
| "environments" | 多少个环境（dev、staging、prod）？环境对等策略？ |
| "rollback" | 回滚策略？蓝绿、金丝雀还是即时回滚？数据库回滚的考量？ |
| "secrets" 或 "config" | 密钥管理——环境变量、vault 还是平台原生？每个环境的配置策略？ |
