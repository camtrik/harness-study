# 用户设置模板

`.planning/phases/XX-name/{phase}-USER-SETUP.md` 的模板——Claude 无法自动化、必须人工完成的配置。

**目的：** 记录确实需要人工操作才能完成的设置任务——账号创建、仪表盘配置、密钥获取。Claude 自动化了所有可能的部分；此文件仅记录剩下的部分。

---

## 文件模板

```markdown
# 阶段 {X}：需要用户设置

**生成日期：** [YYYY-MM-DD]
**阶段：** {phase-name}
**状态：** 未完成

完成以下项目以使集成正常运行。Claude 已自动化了所有可能的部分；这些项目需要人工访问外部仪表盘/账号。

## 环境变量

| 状态 | 变量 | 来源 | 添加到 |
|--------|----------|--------|--------|
| [ ] | `ENV_VAR_NAME` | [服务仪表盘 → 路径 → 到 → 值] | `.env.local` |
| [ ] | `ANOTHER_VAR` | [服务仪表盘 → 路径 → 到 → 值] | `.env.local` |

## 账号设置

[仅当需要创建新账号时]

- [ ] **创建 [服务] 账号**
  - URL：[注册 URL]
  - 可跳过：如果已有账号

## 仪表盘配置

[仅当需要仪表盘配置时]

- [ ] **[配置任务]**
  - 位置：[服务仪表盘 → 路径 → 到 → 设置]
  - 设为：[所需值或配置]
  - 备注：[任何重要详情]

## 验证

完成设置后，使用以下命令验证：

```bash
# [验证命令]
```

预期结果：
- [成功的标志]

---

**所有项目完成后：** 将文件顶部的状态标记为"已完成"。
```

---

## 何时生成

当计划 frontmatter 包含 `user_setup` 字段时，生成 `{phase}-USER-SETUP.md`。

**触发条件：** PLAN.md frontmatter 中存在 `user_setup` 且有项目。

**位置：** 与 PLAN.md 和 SUMMARY.md 相同的目录。

**时机：** 在 execute-plan.md 中任务完成后、SUMMARY.md 创建前生成。

---

## Frontmatter 模式

在 PLAN.md 中，`user_setup` 声明需要人工配置的内容：

```yaml
user_setup:
  - service: stripe
    why: "支付处理需要 API 密钥"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Stripe Dashboard → Developers → API keys → Secret key"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Stripe Dashboard → Developers → Webhooks → Signing secret"
    dashboard_config:
      - task: "创建 webhook 端点"
        location: "Stripe Dashboard → Developers → Webhooks → Add endpoint"
        details: "URL: https://[your-domain]/api/webhooks/stripe, Events: checkout.session.completed, customer.subscription.*"
    local_dev:
      - "运行：stripe listen --forward-to localhost:3000/api/webhooks/stripe"
      - "使用 CLI 输出的 webhook secret 进行本地测试"
```

---

## 自动化优先规则

**USER-SETUP.md 仅包含 Claude 确实无法做到的事。**

| Claude 可以做到（不在 USER-SETUP 内） | Claude 无法做到（→ USER-SETUP） |
|-----------------------------------|--------------------------------|
| `npm install stripe` | 创建 Stripe 账号 |
| 编写 webhook 处理代码 | 从仪表盘获取 API 密钥 |
| 创建 `.env.local` 文件结构 | 复制实际的密钥值 |
| 运行 `stripe listen` | 认证 Stripe CLI（浏览器 OAuth） |
| 配置 package.json | 访问外部服务仪表盘 |
| 编写任何代码 | 从第三方系统获取密钥 |

**判断标准：** "这需要人工在浏览器中，访问一个 Claude 没有凭证的账号吗？"
- 是 → USER-SETUP.md
- 否 → Claude 自动完成

---

## 服务特定示例

<stripe_example>
```markdown
# 阶段 10：需要用户设置

**生成日期：** 2025-01-14
**阶段：** 10-monetization
**状态：** 未完成

完成以下项目以使 Stripe 集成正常运行。

## 环境变量

| 状态 | 变量 | 来源 | 添加到 |
|--------|----------|--------|--------|
| [ ] | `STRIPE_SECRET_KEY` | Stripe Dashboard → Developers → API keys → Secret key | `.env.local` |
| [ ] | `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe Dashboard → Developers → API keys → Publishable key | `.env.local` |
| [ ] | `STRIPE_WEBHOOK_SECRET` | Stripe Dashboard → Developers → Webhooks → [endpoint] → Signing secret | `.env.local` |

## 账号设置

- [ ] **创建 Stripe 账号**（如需）
  - URL：https://dashboard.stripe.com/register
  - 可跳过：如果已有 Stripe 账号

## 仪表盘配置

- [ ] **创建 webhook 端点**
  - 位置：Stripe Dashboard → Developers → Webhooks → Add endpoint
  - 端点 URL：`https://[your-domain]/api/webhooks/stripe`
  - 要发送的事件：
    - `checkout.session.completed`
    - `customer.subscription.created`
    - `customer.subscription.updated`
    - `customer.subscription.deleted`

- [ ] **创建产品和价格**（如果使用订阅层级）
  - 位置：Stripe Dashboard → Products → Add product
  - 创建每个订阅层级
  - 复制 Price ID 到：
    - `STRIPE_STARTER_PRICE_ID`
    - `STRIPE_PRO_PRICE_ID`

## 本地开发

本地 webhook 测试：
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
使用 CLI 输出的 webhook signing secret（以 `whsec_` 开头）。

## 验证

完成设置后：

```bash
# 检查环境变量是否设置
grep STRIPE .env.local

# 验证构建通过
npm run build

# 测试 webhook 端点（应返回 400 bad signature，而非 500 crash）
curl -X POST http://localhost:3000/api/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{}'
```

预期：构建通过，webhook 返回 400（签名验证正常工作）。

---

**所有项目完成后：** 将文件顶部的状态标记为"已完成"。
```
</stripe_example>

<supabase_example>
```markdown
# 阶段 2：需要用户设置

**生成日期：** 2025-01-14
**阶段：** 02-authentication
**状态：** 未完成

完成以下项目以使 Supabase Auth 正常运行。

## 环境变量

| 状态 | 变量 | 来源 | 添加到 |
|--------|----------|--------|--------|
| [ ] | `NEXT_PUBLIC_SUPABASE_URL` | Supabase Dashboard → Settings → API → Project URL | `.env.local` |
| [ ] | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase Dashboard → Settings → API → anon public | `.env.local` |
| [ ] | `SUPABASE_SERVICE_ROLE_KEY` | Supabase Dashboard → Settings → API → service_role | `.env.local` |

## 账号设置

- [ ] **创建 Supabase 项目**
  - URL：https://supabase.com/dashboard/new
  - 可跳过：如果已有此应用的项目

## 仪表盘配置

- [ ] **启用邮箱认证**
  - 位置：Supabase Dashboard → Authentication → Providers
  - 启用：Email provider
  - 配置：确认邮箱（根据偏好开启/关闭）

- [ ] **配置 OAuth 提供商**（如果使用社交登录）
  - 位置：Supabase Dashboard → Authentication → Providers
  - Google：添加来自 Google Cloud Console 的 Client ID 和 Secret
  - GitHub：添加来自 GitHub OAuth Apps 的 Client ID 和 Secret

## 验证

完成设置后：

```bash
# 检查环境变量
grep SUPABASE .env.local

# 验证连接（在项目目录中运行）
npx supabase status
```

---

**所有项目完成后：** 将文件顶部的状态标记为"已完成"。
```
</supabase_example>

<sendgrid_example>
```markdown
# 阶段 5：需要用户设置

**生成日期：** 2025-01-14
**阶段：** 05-notifications
**状态：** 未完成

完成以下项目以使 SendGrid 邮件正常运行。

## 环境变量

| 状态 | 变量 | 来源 | 添加到 |
|--------|----------|--------|--------|
| [ ] | `SENDGRID_API_KEY` | SendGrid Dashboard → Settings → API Keys → Create API Key | `.env.local` |
| [ ] | `SENDGRID_FROM_EMAIL` | 您已验证的发件人邮箱地址 | `.env.local` |

## 账号设置

- [ ] **创建 SendGrid 账号**
  - URL：https://signup.sendgrid.com/
  - 可跳过：如果已有账号

## 仪表盘配置

- [ ] **验证发件人身份**
  - 位置：SendGrid Dashboard → Settings → Sender Authentication
  - 选项 1：Single Sender Verification（快速，用于开发）
  - 选项 2：Domain Authentication（生产环境）

- [ ] **创建 API Key**
  - 位置：SendGrid Dashboard → Settings → API Keys → Create API Key
  - 权限：Restricted Access → Mail Send（Full Access）
  - 立即复制密钥（仅显示一次）

## 验证

完成设置后：

```bash
# 检查环境变量
grep SENDGRID .env.local

# 测试邮件发送（替换为您的测试邮箱）
curl -X POST http://localhost:3000/api/test-email \
  -H "Content-Type: application/json" \
  -d '{"to": "your@email.com"}'
```

---

**所有项目完成后：** 将文件顶部的状态标记为"已完成"。
```
</sendgrid_example>

---

## 指南

**绝不要包含：** 实际的密钥值。Claude 可以自动完成的步骤（包安装、代码变更）。

**命名：** `{phase}-USER-SETUP.md` 匹配阶段编号模式。
**状态跟踪：** 用户标记复选框并在完成时更新状态行。
**可搜索性：** `grep -r "USER-SETUP" .planning/` 找到所有有用户需求的阶段。
