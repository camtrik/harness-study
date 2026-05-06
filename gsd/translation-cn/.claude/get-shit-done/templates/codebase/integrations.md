# 外部集成模板

`.planning/codebase/INTEGRATIONS.md` 的模板——记录外部服务依赖。

**目的：** 记录此代码库与哪些外部系统通信。聚焦于"在我们的代码之外我们依赖什么"。

---

## 文件模板

```markdown
# 外部集成

**分析日期：** [YYYY-MM-DD]

## API 与外部服务

**支付处理：**
- [服务] - [用途：例如"订阅计费、一次性支付"]
  - SDK/客户端：[例如"stripe npm package v14.x"]
  - 认证：[例如"API 密钥在 STRIPE_SECRET_KEY 环境变量中"]
  - 使用的端点：[例如"结账会话、webhooks"]

**邮件/SMS：**
- [服务] - [用途：例如"事务性邮件"]
  - SDK/客户端：[例如"sendgrid/mail v8.x"]
  - 认证：[例如"API 密钥在 SENDGRID_API_KEY 环境变量中"]
  - 模板：[例如"在 SendGrid 仪表盘管理"]

**外部 API：**
- [服务] - [用途]
  - 集成方法：[例如"通过 fetch 的 REST API"、"GraphQL 客户端"]
  - 认证：[例如"OAuth2 token 在 AUTH_TOKEN 环境变量中"]
  - 速率限制：[如适用]

## 数据存储

**数据库：**
- [类型/提供商] - [例如"Supabase 上的 PostgreSQL"]
  - 连接：[例如"通过 DATABASE_URL 环境变量"]
  - 客户端：[例如"Prisma ORM v5.x"]
  - 迁移：[例如"prisma migrate 在 migrations/ 中"]

**文件存储：**
- [服务] - [例如"AWS S3 用于用户上传"]
  - SDK/客户端：[例如"@aws-sdk/client-s3"]
  - 认证：[例如"IAM 凭证在 AWS_* 环境变量中"]
  - 桶：[例如"prod-uploads、dev-uploads"]

**缓存：**
- [服务] - [例如"Redis 用于会话存储"]
  - 连接：[例如"REDIS_URL 环境变量"]
  - 客户端：[例如"ioredis v5.x"]

## 认证与身份

**认证提供商：**
- [服务] - [例如"Supabase Auth"、"Auth0"、"自定义 JWT"]
  - 实现：[例如"Supabase 客户端 SDK"]
  - Token 存储：[例如"httpOnly cookies"、"localStorage"]
  - 会话管理：[例如"JWT refresh tokens"]

**OAuth 集成：**
- [提供商] - [例如"Google OAuth 用于登录"]
  - 凭证：[例如"GOOGLE_CLIENT_ID、GOOGLE_CLIENT_SECRET"]
  - 范围：[例如"email、profile"]

## 监控与可观测性

**错误跟踪：**
- [服务] - [例如"Sentry"]
  - DSN：[例如"SENTRY_DSN 环境变量"]
  - 发布跟踪：[例如"通过 SENTRY_RELEASE"]

**分析：**
- [服务] - [例如"Mixpanel 用于产品分析"]
  - Token：[例如"MIXPANEL_TOKEN 环境变量"]
  - 跟踪的事件：[例如"用户操作、页面浏览"]

**日志：**
- [服务] - [例如"CloudWatch"、"Datadog"、"无（仅 stdout）"]
  - 集成：[例如"AWS Lambda 内置"]

## CI/CD 与部署

**托管：**
- [平台] - [例如"Vercel"、"AWS Lambda"、"ECS 上的 Docker"]
  - 部署：[例如"推送到 main 分支时自动部署"]
  - 环境变量：[例如"在 Vercel 仪表盘配置"]

**CI 流水线：**
- [服务] - [例如"GitHub Actions"]
  - 工作流：[例如"test.yml、deploy.yml"]
  - 密钥：[例如"存储在 GitHub 仓库 secrets 中"]

## 环境配置

**开发环境：**
- 必需的环境变量：[列出关键变量]
- 密钥位置：[例如".env.local（gitignored）"、"1Password vault"]
- Mock/桩服务：[例如"Stripe test mode"、"本地 PostgreSQL"]

**预发布环境：**
- 环境特定差异：[例如"使用预发布 Stripe 账号"]
- 数据：[例如"单独的预发布数据库"]

**生产环境：**
- 密钥管理：[例如"Vercel 环境变量"]
- 故障转移/冗余：[例如"多区域数据库复制"]

## Webhooks 与回调

**入站：**
- [服务] - [端点：例如"/api/webhooks/stripe"]
  - 验证：[例如"通过 stripe.webhooks.constructEvent 验证签名"]
  - 事件：[例如"payment_intent.succeeded、customer.subscription.updated"]

**出站：**
- [服务] - [触发条件]
  - 端点：[例如"用户注册时调用外部 CRM webhook"]
  - 重试逻辑：[如适用]

---

*集成审计：[日期]*
*添加/移除外部服务时更新*
```

<good_examples>
```markdown
# 外部集成

**分析日期：** 2025-01-20

## API 与外部服务

**支付处理：**
- Stripe - 订阅计费和一次性课程支付
  - SDK/客户端：stripe npm package v14.8
  - 认证：API 密钥在 STRIPE_SECRET_KEY 环境变量
  - 使用的端点：结账会话、客户门户、webhooks

**邮件/SMS：**
- SendGrid - 事务性邮件（收据、密码重置）
  - SDK/客户端：@sendgrid/mail v8.1
  - 认证：API 密钥在 SENDGRID_API_KEY 环境变量
  - 模板：在 SendGrid 仪表盘管理（代码中存储模板 ID）

**外部 API：**
- OpenAI API - 课程内容生成
  - 集成方法：通过 openai npm package v4.x 的 REST API
  - 认证：Bearer token 在 OPENAI_API_KEY 环境变量
  - 速率限制：3500 请求/分钟（tier 3）

## 数据存储

**数据库：**
- Supabase 上的 PostgreSQL - 主数据存储
  - 连接：通过 DATABASE_URL 环境变量
  - 客户端：Prisma ORM v5.8
  - 迁移：prisma migrate 在 prisma/migrations/

**文件存储：**
- Supabase Storage - 用户上传（头像、课程资料）
  - SDK/客户端：@supabase/supabase-js v2.x
  - 认证：服务角色密钥在 SUPABASE_SERVICE_ROLE_KEY
  - 桶：avatars（公开）、course-materials（私有）

**缓存：**
- 当前无（所有数据库查询，无 Redis）

## 认证与身份

**认证提供商：**
- Supabase Auth - 邮箱/密码 + OAuth
  - 实现：Supabase 客户端 SDK，服务器端会话管理
  - Token 存储：通过 @supabase/ssr 的 httpOnly cookies
  - 会话管理：由 Supabase 处理 JWT refresh tokens

**OAuth 集成：**
- Google OAuth - 社交登录
  - 凭证：GOOGLE_CLIENT_ID、GOOGLE_CLIENT_SECRET（Supabase 仪表盘）
  - 范围：email、profile

## 监控与可观测性

**错误跟踪：**
- Sentry - 服务器和客户端错误
  - DSN：SENTRY_DSN 环境变量
  - 发布跟踪：通过 SENTRY_RELEASE 跟踪 Git commit SHA

**分析：**
- 无（计划：Mixpanel）

**日志：**
- Vercel logs - 仅 stdout/stderr
  - 保留期限：Pro 计划 7 天

## CI/CD 与部署

**托管：**
- Vercel - Next.js 应用托管
  - 部署：推送到 main 分支时自动部署
  - 环境变量：在 Vercel 仪表盘配置（同步到 .env.example）

**CI 流水线：**
- GitHub Actions - 测试和类型检查
  - 工作流：.github/workflows/ci.yml
  - 密钥：无（公开仓库仅运行测试）

## 环境配置

**开发环境：**
- 必需的环境变量：DATABASE_URL、NEXT_PUBLIC_SUPABASE_URL、NEXT_PUBLIC_SUPABASE_ANON_KEY
- 密钥位置：.env.local（gitignored），团队通过 1Password vault 共享
- Mock/桩服务：Stripe test mode、Supabase 本地开发项目

**预发布环境：**
- 使用单独的 Supabase 预发布项目
- Stripe test mode
- 同一 Vercel 账号，不同环境

**生产环境：**
- 密钥管理：Vercel 环境变量
- 数据库：Supabase 生产项目，每日备份

## Webhooks 与回调

**入站：**
- Stripe - /api/webhooks/stripe
  - 验证：通过 stripe.webhooks.constructEvent 进行签名验证
  - 事件：payment_intent.succeeded、customer.subscription.updated、customer.subscription.deleted

**出站：**
- 无

---

*集成审计：2025-01-20*
*添加/移除外部服务时更新*
```
</good_examples>

<guidelines>
**INTEGRATIONS.md 应包含的内容：**
- 代码与之通信的外部服务
- 认证模式（密钥存储位置，而非密钥本身）
- 使用的 SDK 和客户端库
- 环境变量名（而非值）
- Webhook 端点和验证方法
- 数据库连接模式
- 文件存储位置
- 监控和日志服务

**不应包含的内容：**
- 实际的 API 密钥或密钥（绝不要写这些）
- 内部架构（那是 ARCHITECTURE.md 的事）
- 代码模式（那是 PATTERNS.md 的事）
- 技术选择（那是 STACK.md 的事）
- 性能问题（那是 CONCERNS.md 的事）

**填充此模板时：**
- 检查 .env.example 或 .env.template 中的必需环境变量
- 寻找 SDK 导入（stripe、@sendgrid/mail 等）
- 检查路由/端点中的 webhook 处理函数
- 记录密钥的管理位置（而非密钥本身）
- 记录环境特定差异（dev/staging/prod）
- 包含每个服务的认证模式

**在以下场景中对阶段规划有用：**
- 添加新的外部服务集成
- 调试认证问题
- 理解应用外部的数据流
- 设置新环境
- 审计第三方依赖
- 为服务中断或迁移做规划

**安全说明：**
记录密钥存储的位置（环境变量、Vercel 仪表盘、1Password），绝不要记录密钥是什么。
</guidelines>
