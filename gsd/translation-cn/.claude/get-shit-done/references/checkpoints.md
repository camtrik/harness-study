<overview>
计划自主执行。检查点是将需要人类验证或决策的交互点正式化的机制。

**核心原则：** Claude 通过 CLI/API 自动化一切。检查点用于验证和决策，而非手动工作。

**黄金规则：**
1. **Claude 能运行的就由 Claude 运行**——绝不让用户执行 CLI 命令、启动服务器或运行构建
2. **Claude 设置验证环境**——启动开发服务器、填充数据库、配置环境变量
3. **用户只做需要人类判断的事情**——视觉检查、UX 评估、"这感觉对吗？"
4. **密钥来自用户，自动化来自 Claude**——请求 API key，然后 Claude 通过 CLI 使用它们
5. **Auto-mode 绕过验证/决策检查点**——当配置中 `workflow._auto_chain_active` 或 `workflow.auto_advance` 为 true 时：human-verify 自动批准，decision 自动选择第一个选项，human-action 仍然停止（认证门无法自动化）
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify（最常见 - 90%）

**时机：** Claude 完成自动化工作，人类确认其工作正确。

**用于：**
- 可视化 UI 检查（布局、样式、响应性）
- 交互流程（点击向导、测试用户流）
- 功能验证（功能按预期工作）
- 音频/视频播放质量
- 动画流畅度
- 无障碍测试

**结构：**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[Claude 自动化并部署/构建的内容]</what-built>
  <how-to-verify>
    [精确的测试步骤——URL、命令、预期行为]
  </how-to-verify>
  <resume-signal>[如何继续——"approved"、"yes"、或描述问题]</resume-signal>
</task>
```

**示例：UI 组件（展示关键模式：Claude 在检查点之前启动服务器）**
```xml
<task type="auto">
  <name>构建响应式仪表盘布局</name>
  <files>src/components/Dashboard.tsx, src/app/dashboard/page.tsx</files>
  <action>创建包含侧边栏、头部和内容区域的仪表盘。使用 Tailwind 响应式类适配移动端。</action>
  <verify>npm run build 成功，无 TypeScript 错误</verify>
  <done>仪表盘组件构建无错误</done>
</task>

<task type="auto">
  <name>启动开发服务器以进行验证</name>
  <action>在后台运行 `npm run dev`，等待 "ready" 消息，捕获端口</action>
  <verify>fetch http://localhost:3000 返回 200</verify>
  <done>开发服务器运行在 http://localhost:3000</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>响应式仪表盘布局——开发服务器运行在 http://localhost:3000</what-built>
  <how-to-verify>
    访问 http://localhost:3000/dashboard 并验证：
    1. 桌面端（>1024px）：侧边栏在左，内容在右，头部在顶部
    2. 平板端（768px）：侧边栏折叠为汉堡菜单
    3. 手机端（375px）：单列布局，底部导航出现
    4. 任何尺寸下无布局跳动或水平滚动
  </how-to-verify>
  <resume-signal>输入 "approved" 或描述布局问题</resume-signal>
</task>
```

**示例：Xcode 构建**
```xml
<task type="auto">
  <name>使用 Xcode 构建 macOS 应用</name>
  <files>App.xcodeproj, Sources/</files>
  <action>运行 `xcodebuild -project App.xcodeproj -scheme App build`。检查输出中的编译错误。</action>
  <verify>构建输出包含 "BUILD SUCCEEDED"，无错误</verify>
  <done>应用构建成功</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>在 DerivedData/Build/Products/Debug/App.app 构建的 macOS 应用</what-built>
  <how-to-verify>
    打开 App.app 并测试：
    - 应用启动无崩溃
    - 菜单栏图标出现
    - 偏好设置窗口正常打开
    - 无视觉故障或布局问题
  </how-to-verify>
  <resume-signal>输入 "approved" 或描述问题</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision（9%）

**时机：** 人类必须做出影响实现方向的选择。

**用于：**
- 技术选型（哪个认证提供商、哪个数据库）
- 架构决策（monorepo 还是分离仓库）
- 设计选择（配色方案、布局方式）
- 功能优先级（构建哪个变体）
- 数据模型决策（模式结构）

**结构：**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[正在决策的内容]</decision>
  <context>[为什么此决策很重要]</context>
  <options>
    <option id="option-a">
      <name>[选项名称]</name>
      <pros>[优势]</pros>
      <cons>[权衡]</cons>
    </option>
    <option id="option-b">
      <name>[选项名称]</name>
      <pros>[优势]</pros>
      <cons>[权衡]</cons>
    </option>
  </options>
  <resume-signal>[如何表示选择]</resume-signal>
</task>
```

**示例：认证提供商选择**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>选择认证提供商</decision>
  <context>
    应用需要用户认证。三个可靠选项各有不同的权衡。
  </context>
  <options>
    <option id="supabase">
      <name>Supabase Auth</name>
      <pros>与我们使用的 Supabase DB 内置集成，慷慨的免费额度，行级安全集成</pros>
      <cons>UI 可定制性较低，绑定到 Supabase 生态系统</cons>
    </option>
    <option id="clerk">
      <name>Clerk</name>
      <pros>漂亮的预构建 UI，最佳开发者体验，优秀的文档</pros>
      <cons>10k MAU 后开始收费，供应商锁定</cons>
    </option>
    <option id="nextauth">
      <name>NextAuth.js</name>
      <pros>免费、自托管、最大控制权、广泛采用</pros>
      <cons>更多设置工作，需要自行管理安全更新，UI 需要自行构建</cons>
    </option>
  </options>
  <resume-signal>选择：supabase、clerk 或 nextauth</resume-signal>
</task>
```

**示例：数据库选择**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>为用户数据选择数据库</decision>
  <context>
    应用需要为用户、会话和用户生成的内容提供持久存储。
    预期规模：第一年 10k 用户、1M 条记录。
  </context>
  <options>
    <option id="supabase">
      <name>Supabase（Postgres）</name>
      <pros>完整 SQL、慷慨的免费额度、内置认证、实时订阅</pros>
      <cons>实时功能的供应商锁定，灵活性不如原生 Postgres</cons>
    </option>
    <option id="planetscale">
      <name>PlanetScale（MySQL）</name>
      <pros>无服务器扩展、分支工作流、优秀的 DX</pros>
      <cons>MySQL 而非 Postgres，免费版无外键</cons>
    </option>
    <option id="convex">
      <name>Convex</name>
      <pros>默认实时、TypeScript 原生、自动缓存</pros>
      <cons>较新的平台、不同的思维模型、SQL 灵活性较低</cons>
    </option>
  </options>
  <resume-signal>选择：supabase、planetscale 或 convex</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action（1% - 罕见）

**时机：** 操作无法通过 CLI/API 完成且需要人工交互时，或 Claude 在自动化过程中遇到认证阻碍。

**仅用于：**
- **认证阻碍**——Claude 尝试了 CLI/API 但需要凭证（这不是失败）
- 邮件验证链接（点击邮件中的链接）
- 短信双因素验证码（手机验证）
- 手动账户审批（平台需要人工审核）
- 信用卡 3D Secure 流程（基于网页的支付授权）
- OAuth 应用审批（基于网页的审批）

**不要用于预先计划的手动工作：**
- 部署（使用 CLI——必要时标记为认证阻碍）
- 创建 webhook/数据库（使用 API/CLI——必要时标记为认证阻碍）
- 运行构建/测试（使用 Bash 工具）
- 创建文件（使用 Write 工具）

**结构：**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[人类必须做的事情——Claude 已完成所有可自动化的]</action>
  <instructions>
    [Claude 已完成自动化的工作]
    [需要人工操作的 ONE 件事]
  </instructions>
  <verification>[Claude 之后可以检查什么]</verification>
  <resume-signal>[如何继续]</resume-signal>
</task>
```

**示例：邮件验证**
```xml
<task type="auto">
  <name>通过 API 创建 SendGrid 账户</name>
  <action>使用 SendGrid API 创建具有提供邮箱的子用户账户。请求验证邮件。</action>
  <verify>API 返回 201，账户已创建</verify>
  <done>账户已创建，验证邮件已发送</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>完成 SendGrid 账户的邮件验证</action>
  <instructions>
    我创建了账户并请求了验证邮件。
    请检查你的收件箱中的 SendGrid 验证链接并点击它。
  </instructions>
  <verification>SendGrid API key 有效：curl 测试成功</verification>
  <resume-signal>邮件验证完成后输入 "done"</resume-signal>
</task>
```

**示例：认证阻碍（动态检查点）**
```xml
<task type="auto">
  <name>部署到 Vercel</name>
  <files>.vercel/, vercel.json</files>
  <action>运行 `vercel --yes` 进行部署</action>
  <verify>vercel ls 显示部署，fetch 返回 200</verify>
</task>

<!-- 如果 vercel 返回 "Error: Not authenticated"，Claude 即时动态创建检查点 -->

<task type="checkpoint:human-action" gate="blocking">
  <action>认证 Vercel CLI 以便继续部署</action>
  <instructions>
    我尝试部署但收到认证错误。
    请运行：vercel login
    这将打开你的浏览器——完成认证流程。
  </instructions>
  <verification>vercel whoami 返回你的账户邮箱</verification>
  <resume-signal>认证完成后输入 "done"</resume-signal>
</task>

<!-- 认证后，Claude 重试部署 -->

<task type="auto">
  <name>重试 Vercel 部署</name>
  <action>运行 `vercel --yes`（现在已认证）</action>
  <verify>vercel ls 显示部署，fetch 返回 200</verify>
</task>
```

**关键区别：** 认证阻碍是在 Claude 遇到认证错误时动态创建的。不是预先计划的——Claude 先自动化，仅在受阻时才请求凭证。
</type>
</checkpoint_types>

<execution_protocol>

当 Claude 遇到 `type="checkpoint:*"` 时：

1. **立即停止**——不继续下一个任务
2. **清晰显示检查点**使用下面的格式
3. **等待用户响应**——不要虚构完成
4. **尽可能验证**——检查文件、运行测试，执行指定的任何操作
5. **恢复执行**——仅在确认后继续下一个任务

**对于 checkpoint:human-verify：**
```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: 需要验证                                        ║
╚══════════════════════════════════════════════════════════════╝

进度：5/8 任务完成
任务：响应式仪表盘布局

已构建：响应式仪表盘，位于 /dashboard

如何验证：
  1. 访问：http://localhost:3000/dashboard
  2. 桌面端（>1024px）：侧边栏可见，内容填充剩余空间
  3. 平板端（768px）：侧边栏折叠为图标
  4. 手机端（375px）：侧边栏隐藏，汉堡菜单出现

──────────────────────────────────────────────────────────────
→ 你的操作：输入 "approved" 或描述问题
──────────────────────────────────────────────────────────────
```

**对于 checkpoint:decision：**
```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: 需要决策                                        ║
╚══════════════════════════════════════════════════════════════╝

进度：2/6 任务完成
任务：选择认证提供商

决策：我们应该使用哪个认证提供商？

上下文：需要用户认证。三个选项各有不同的权衡。

选项：
  1. supabase - 与我们的数据库内置集成，免费额度
     优势：行级安全集成，慷慨的免费额度
     劣势：UI 可定制性较低，生态系统锁定

  2. clerk - 最佳 DX，10k 用户后付费
     优势：漂亮的预构建 UI，优秀的文档
     劣势：供应商锁定，规模化后定价

  3. nextauth - 自托管，最大控制权
     优势：免费，无供应商锁定，广泛采用
     劣势：更多设置工作，需自行管理安全更新

──────────────────────────────────────────────────────────────
→ 你的操作：选择 supabase、clerk 或 nextauth
──────────────────────────────────────────────────────────────
```

**对于 checkpoint:human-action：**
```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: 需要操作                                        ║
╚══════════════════════════════════════════════════════════════╝

进度：3/8 任务完成
任务：部署到 Vercel

尝试：vercel --yes
错误：未认证。请运行 'vercel login'

你需要做的：
  1. 运行：vercel login
  2. 当它打开浏览器时完成认证
  3. 完成后回到这里

我将验证：vercel whoami 返回你的账户

──────────────────────────────────────────────────────────────
→ 你的操作：认证完成后输入 "done"
──────────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**认证阻碍 = Claude 尝试了 CLI/API，遇到认证错误。** 这不是失败——是需要人工输入来解开的阻碍。

**模式：** Claude 尝试自动化 → 认证错误 → 创建 checkpoint:human-action → 用户认证 → Claude 重试 → 继续

**阻碍协议：**
1. 认识到这不是失败——缺少认证是预期的
2. 停止当前任务——不要反复重试
3. 动态创建 checkpoint:human-action
4. 提供精确的认证步骤
5. 验证认证有效
6. 重试原始任务
7. 正常继续

**关键区别：**
- 预先计划的检查点："我需要你做 X"（错误——Claude 应该自动化）
- 认证阻碍："我尝试自动化 X 但需要凭证"（正确——解除自动化阻碍）

</authentication_gates>

<automation_reference>

**规则：** 如果有 CLI/API 可用，Claude 就执行。绝不让人类执行可自动化的工作。

## 服务 CLI 参考

| 服务 | CLI/API | 关键命令 | 认证阻碍 |
|---------|---------|--------------|-----------|
| Vercel | `vercel` | `--yes`、`env add`、`--prod`、`ls` | `vercel login` |
| Railway | `railway` | `init`、`up`、`variables set` | `railway login` |
| Fly | `fly` | `launch`、`deploy`、`secrets set` | `fly auth login` |
| Stripe | `stripe` + API | `listen`、`trigger`、API 调用 | .env 中的 API key |
| Supabase | `supabase` | `init`、`link`、`db push`、`gen types` | `supabase login` |
| Upstash | `upstash` | `redis create`、`redis get` | `upstash auth login` |
| PlanetScale | `pscale` | `database create`、`branch create` | `pscale auth login` |
| GitHub | `gh` | `repo create`、`pr create`、`secret set` | `gh auth login` |
| Node | `npm`/`pnpm` | `install`、`run build`、`test`、`run dev` | N/A |
| Xcode | `xcodebuild` | `-project`、`-scheme`、`build`、`test` | N/A |
| Convex | `npx convex` | `dev`、`deploy`、`env set`、`env get` | `npx convex login` |

## 环境变量自动化

**环境变量文件：** 使用 Write/Edit 工具。绝不让人类手动创建 .env。

**通过 CLI 设置仪表盘环境变量：**

| 平台 | CLI 命令 | 示例 |
|----------|-------------|---------|
| Convex | `npx convex env set` | `npx convex env set OPENAI_API_KEY sk-...` |
| Vercel | `vercel env add` | `vercel env add STRIPE_KEY production` |
| Railway | `railway variables set` | `railway variables set API_KEY=value` |
| Fly | `fly secrets set` | `fly secrets set DATABASE_URL=...` |
| Supabase | `supabase secrets set` | `supabase secrets set MY_SECRET=value` |

**密钥收集模式：**
```xml
<!-- 错误：要求用户在仪表盘中添加环境变量 -->
<task type="checkpoint:human-action">
  <action>将 OPENAI_API_KEY 添加到 Convex 仪表盘</action>
  <instructions>前往 dashboard.convex.dev → 设置 → 环境变量 → 添加</instructions>
</task>

<!-- 正确：Claude 请求值，然后通过 CLI 添加 -->
<task type="checkpoint:human-action">
  <action>提供你的 OpenAI API key</action>
  <instructions>
    Convex 后端需要你的 OpenAI API key。
    从 https://platform.openai.com/api-keys 获取。
    粘贴 key（以 sk- 开头）
  </instructions>
  <verification>我将通过 `npx convex env set` 添加并验证</verification>
  <resume-signal>粘贴你的 API key</resume-signal>
</task>

<task type="auto">
  <name>在 Convex 中配置 OpenAI key</name>
  <action>运行 `npx convex env set OPENAI_API_KEY {用户提供的 key}`</action>
  <verify>`npx convex env get OPENAI_API_KEY` 返回 key（掩码后）</verify>
</task>
```

## 开发服务器自动化

| 框架 | 启动命令 | 就绪信号 | 默认 URL |
|-----------|---------------|--------------|-------------|
| Next.js | `npm run dev` | "Ready in" 或 "started server" | http://localhost:3000 |
| Vite | `npm run dev` | "ready in" | http://localhost:5173 |
| Convex | `npx convex dev` | "Convex functions ready" | N/A（仅后端） |
| Express | `npm start` | "listening on port" | http://localhost:3000 |
| Django | `python manage.py runserver` | "Starting development server" | http://localhost:8000 |

**服务器生命周期：**
```bash
# 在后台运行，捕获 PID
npm run dev &
DEV_SERVER_PID=$!

# 等待就绪（最长 30 秒）——使用 fetch() 实现跨平台兼容
timeout 30 bash -c 'until node -e "fetch(\"http://localhost:3000\").then(r=>{process.exit(r.ok?0:1)}).catch(()=>process.exit(1))" 2>/dev/null; do sleep 1; done'
```

**端口冲突：** 杀死僵尸进程（`lsof -ti:3000 | xargs kill`）或使用备用端口（`--port 3001`）。

**服务器在检查点期间保持运行。** 仅在计划完成、切换到生产环境或端口需要用于其他服务时关闭。

## CLI 安装处理

| CLI | 自动安装？ | 命令 |
|-----|---------------|---------|
| npm/pnpm/yarn | 否——询问用户 | 用户选择包管理器 |
| vercel | 是 | `npm i -g vercel` |
| gh（GitHub） | 是 | `brew install gh`（macOS）或 `apt install gh`（Linux） |
| stripe | 是 | `npm i -g stripe` |
| supabase | 是 | `npm i -g supabase` |
| convex | 否——使用 npx | `npx convex`（无需安装） |
| fly | 是 | `brew install flyctl` 或 curl 安装器 |
| railway | 是 | `npm i -g @railway/cli` |

**协议：** 尝试命令 → "command not found" → 可自动安装？→ 是：静默安装，重试 → 否：检查点要求用户安装。

## 检查点之前的自动化失败处理

| 失败 | 响应 |
|---------|----------|
| 服务器无法启动 | 检查错误，修复问题，重试（不要继续到检查点） |
| 端口被占用 | 杀死僵尸进程或使用备用端口 |
| 缺少依赖 | 运行 `npm install`，重试 |
| 构建错误 | 先修复错误（这是 bug，不是检查点问题） |
| 认证错误 | 创建认证阻碍检查点 |
| 网络超时 | 按退避策略重试，持续失败则创建检查点 |

**绝不在验证环境损坏时呈现检查点。** 如果本地服务器无响应，不要要求用户"访问 localhost:3000"。

> **跨平台注意：** 使用 `node -e "fetch('http://localhost:3000').then(r=>console.log(r.status))"` 替代 `curl` 进行健康检查。`curl` 在 Windows MSYS/Git Bash 上因 SSL/路径处理问题而不可靠。

## 自动化快速参考

| 操作 | 可自动化？ | Claude 做吗？ |
|--------|--------------|-----------------|
| 部署到 Vercel | 是（`vercel`） | 是 |
| 创建 Stripe webhook | 是（API） | 是 |
| 写入 .env 文件 | 是（Write 工具） | 是 |
| 创建 Upstash DB | 是（`upstash`） | 是 |
| 运行测试 | 是（`npm test`） | 是 |
| 启动开发服务器 | 是（`npm run dev`） | 是 |
| 向 Convex 添加环境变量 | 是（`npx convex env set`） | 是 |
| 向 Vercel 添加环境变量 | 是（`vercel env add`） | 是 |
| 填充数据库 | 是（CLI/API） | 是 |
| 点击邮件验证链接 | 否 | 否 |
| 输入信用卡（3DS） | 否 | 否 |
| 在浏览器中完成 OAuth | 否 | 否 |
| 视觉验证 UI 是否正确 | 否 | 否 |
| 测试交互式用户流 | 否 | 否 |

</automation_reference>

<writing_guidelines>

**要做：**
- 在检查点之前用 CLI/API 自动化一切
- 具体描述："访问 https://myapp.vercel.app"，而不是"检查部署"
- 编号验证步骤
- 说明预期结果："你应该看到 X"
- 提供上下文：为什么需要这个检查点

**不要：**
- 让人类做 Claude 能自动化的工作 ❌
- 假定知识："照常配置设置" ❌
- 跳过步骤："设置数据库"（太模糊）❌
- 在一个检查点中混合多个验证 ❌

**位置：**
- **在自动化完成之后**——不是在 Claude 工作之前
- **在 UI 构建完成之后**——在声明阶段完成之前
- **在依赖工作之前**——决策在实现之前
- **在集成点处**——配置外部服务之后

**错误位置：** 自动化之前 ❌ | 过于频繁 ❌ | 太晚（依赖任务已经需要结果）❌
</writing_guidelines>

<examples>

### 示例 1：数据库设置（无需检查点）

```xml
<task type="auto">
  <name>创建 Upstash Redis 数据库</name>
  <files>.env</files>
  <action>
    1. 运行 `upstash redis create myapp-cache --region us-east-1`
    2. 从输出中捕获连接 URL
    3. 写入到 .env：UPSTASH_REDIS_URL={url}
    4. 用测试命令验证连接
  </action>
  <verify>
    - upstash redis list 显示数据库
    - .env 包含 UPSTASH_REDIS_URL
    - 测试连接成功
  </verify>
  <done>Redis 数据库已创建并配置</done>
</task>

<!-- 无需检查点——Claude 自动完成一切并通过程序验证 -->
```

### 示例 2：完整认证流程（仅在结束时有一个检查点）

```xml
<task type="auto">
  <name>创建用户模式</name>
  <files>src/db/schema.ts</files>
  <action>使用 Drizzle ORM 定义 User、Session、Account 表</action>
  <verify>npm run db:generate 成功</verify>
</task>

<task type="auto">
  <name>创建认证 API 路由</name>
  <files>src/app/api/auth/[...nextauth]/route.ts</files>
  <action>使用 GitHub provider 和 JWT 策略设置 NextAuth</action>
  <verify>TypeScript 编译通过，无错误</verify>
</task>

<task type="auto">
  <name>创建登录 UI</name>
  <files>src/app/login/page.tsx, src/components/LoginButton.tsx</files>
  <action>创建带有 GitHub OAuth 按钮的登录页面</action>
  <verify>npm run build 成功</verify>
</task>

<task type="auto">
  <name>为认证测试启动开发服务器</name>
  <action>在后台运行 `npm run dev`，等待就绪信号</action>
  <verify>fetch http://localhost:3000 返回 200</verify>
  <done>开发服务器运行在 http://localhost:3000</done>
</task>

<!-- 最后只有一个检查点验证完整流程 -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>完整认证流程——开发服务器运行在 http://localhost:3000</what-built>
  <how-to-verify>
    1. 访问：http://localhost:3000/login
    2. 点击 "Sign in with GitHub"
    3. 完成 GitHub OAuth 流程
    4. 验证：重定向到 /dashboard，显示用户名
    5. 刷新页面：会话保持
    6. 点击登出：会话清除
  </how-to-verify>
  <resume-signal>输入 "approved" 或描述问题</resume-signal>
</task>
```
</examples>

<anti_patterns>

### ❌ 坏：要求用户启动开发服务器

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>仪表盘组件</what-built>
  <how-to-verify>
    1. 运行：npm run dev
    2. 访问：http://localhost:3000/dashboard
    3. 检查布局是否正确
  </how-to-verify>
</task>
```

**为什么坏：** Claude 可以运行 `npm run dev`。用户应该只访问 URL，不执行命令。

### ✅ 好：Claude 启动服务器，用户访问

```xml
<task type="auto">
  <name>启动开发服务器</name>
  <action>在后台运行 `npm run dev`</action>
  <verify>fetch http://localhost:3000 返回 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>仪表盘，位于 http://localhost:3000/dashboard（服务器运行中）</what-built>
  <how-to-verify>
    访问 http://localhost:3000/dashboard 并验证：
    1. 布局与设计一致
    2. 无控制台错误
  </how-to-verify>
</task>
```

### ❌ 坏：要求人类部署 / ✅ 好：Claude 自动化

```xml
<!-- 坏：要求用户通过仪表盘部署 -->
<task type="checkpoint:human-action" gate="blocking">
  <action>部署到 Vercel</action>
  <instructions>访问 vercel.com/new → 导入仓库 → 点击部署 → 复制 URL</instructions>
</task>

<!-- 好：Claude 部署，用户验证 -->
<task type="auto">
  <name>部署到 Vercel</name>
  <action>运行 `vercel --yes`。捕获 URL。</action>
  <verify>vercel ls 显示部署，fetch 返回 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>已部署到 {url}</what-built>
  <how-to-verify>访问 {url}，检查首页加载</how-to-verify>
  <resume-signal>输入 "approved"</resume-signal>
</task>
```

### ❌ 坏：检查点过多 / ✅ 好：单个检查点

### ❌ 坏：模糊的验证 / ✅ 好：具体的步骤

### ❌ 坏：要求用户运行 CLI 命令

### ❌ 坏：要求用户在服务之间复制值

</anti_patterns>

<type name="tdd-review">
## checkpoint:tdd-review（仅 TDD 模式）

**时机：** 阶段内所有 wave 完成且启用了 `workflow.tdd_mode`。由 execute-phase 编排器在 `aggregate_results` 之后插入。

**目的：** 协作审查阶段内所有 `type: tdd` 计划的 TDD 门合规性。咨询性质——不阻塞执行。

**用于：**
- 验证每个 TDD 计划的 RED/GREEN/REFACTOR commit 序列
- 呈现门违规（缺少 RED 或 GREEN commit）
- 审查测试质量（测试是否因正确的原因失败）
- 确认最小的 GREEN 实现

**结构：**
```xml
<task type="checkpoint:tdd-review" gate="advisory">
  <what-checked>阶段 {X} 中 {数量} 个计划的 TDD 门合规性</what-checked>
  <gate-results>
    | 计划 | RED | GREEN | REFACTOR | 状态 |
    |------|-----|-------|----------|------|
    | {id} |  ✓  |   ✓   |    ✓     | 通过 |
  </gate-results>
  <violations>[门违规列表，或 "None"]</violations>
  <resume-signal>审查完成——继续到阶段验证</resume-signal>
</task>
```

**Auto-mode 行为：** 当 `workflow._auto_chain_active` 或 `workflow.auto_advance` 为 true 时，TDD 审查检查点自动批准（咨询门——从不阻塞）。
</type>

<summary>

检查点正式化了人机协同的验证和决策点，而非手动工作。

**黄金规则：** 如果 Claude 能自动化，Claude 必须自动化。

**检查点优先级：**
1. **checkpoint:human-verify**（90%）——Claude 自动化一切，人类确认视觉/功能正确性
2. **checkpoint:decision**（9%）——人类做出架构/技术选择
3. **checkpoint:human-action**（1%）——真正不可避免且无 API/CLI 的手动步骤

**何时不使用检查点：**
- Claude 可以通过程序验证的事情（测试、构建）
- 文件操作（Claude 可以读取文件）
- 代码正确性（测试和静态分析）
- 任何可通过 CLI/API 自动化的事情
</summary>
