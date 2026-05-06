# Planner 反模式和具体性示例

> gsd-planner agent 的参考文件。通过 `@` 引用按需加载。
> 对于低于 200K 的上下文窗口，此内容从 agent prompt 中剥离，在此按需加载。

## 检查点反模式

### 坏——要求人类自动化

```xml
<task type="checkpoint:human-action">
  <action>部署到 Vercel</action>
  <instructions>访问 vercel.com，导入仓库，点击部署...</instructions>
</task>
```

**为什么坏：** Vercel 有 CLI。Claude 应该运行 `vercel --yes`。绝不让用户做 Claude 可以通过 CLI/API 自动化的事情。

### 坏——检查点太多

```xml
<task type="auto">创建模式</task>
<task type="checkpoint:human-verify">检查模式</task>
<task type="auto">创建 API</task>
<task type="checkpoint:human-verify">检查 API</task>
```

**为什么坏：** 验证疲劳。不应要求用户验证每个小步骤。在有意义的工作结束时合并为一个检查点。

### 好——单一验证检查点

```xml
<task type="auto">创建模式</task>
<task type="auto">创建 API</task>
<task type="auto">创建 UI</task>
<task type="checkpoint:human-verify">
  <what-built>完整认证流程（模式 + API + UI）</what-built>
  <how-to-verify>测试完整流程：注册、登录、访问受保护页面</how-to-verify>
</task>
```

### 坏——将检查点与实现混合

计划不应将多种检查点类型与实现任务交错。检查点应位于自然的验证边界，而非散布在各处。

## 具体性示例

| 太模糊 | 刚好 |
|-----------|------------|
| "添加认证" | "使用 jose 库添加带刷新轮换的 JWT 认证，存储在 httpOnly cookie 中，15 分钟访问 / 7 天刷新" |
| "创建 API" | "创建 POST /api/projects 端点，接受 {name, description}，验证名称长度 3-50 字符，返回 201 及 project 对象" |
| "美化仪表盘" | "向 Dashboard.tsx 添加 Tailwind 类：网格布局（lg 上 3 列，移动端 1 列），卡片阴影，操作按钮的 hover 状态" |
| "处理错误" | "用 try/catch 包裹 API 调用，在 4xx/5xx 时返回 {error: string}，在客户端通过 sonner 显示 toast" |
| "设置数据库" | "向 schema.prisma 添加 User 和 Project 模型，使用 UUID id、email 唯一约束、createdAt/updatedAt 时间戳、运行 prisma db push" |

**具体性测试：** 另一个 Claude 实例能否在执行任务时无需提出澄清问题？如果不能，添加更多细节。

## 上下文节反模式

### 坏——反射性 SUMMARY 链

```markdown
<context>
@.planning/phases/01-foundation/01-01-SUMMARY.md
@.planning/phases/01-foundation/01-02-SUMMARY.md  <!-- 计划 02 真的需要计划 01 的输出吗？ -->
@.planning/phases/01-foundation/01-03-SUMMARY.md  <!-- 链式增长，上下文膨胀 -->
</context>
```

**为什么坏：** 计划通常是独立的。反射性链式引用（02 引用 01，03 引用 02...）浪费上下文。仅当计划确实使用了来自之前计划的类型/导出或决策影响当前计划时，才引用先前的 SUMMARY 文件。

### 好——选择性上下文

```markdown
<context>
@.planning/PROJECT.md
@.planning/STATE.md
@.planning/phases/01-foundation/01-01-SUMMARY.md  <!-- 使用了计划 01 中定义的 User 类型 -->
</context>
```

## 范围缩减反模式

**任务 action 中禁止的措辞：**
- "v1"、"v2"、"简化版"、"暂时静态"、"暂时硬编码"
- "未来的改进"、"占位符"、"基础版"、"极简实现"
- "稍后接入"、"未来阶段动态化"、"暂时跳过"

如果 CONTEXT.md 的决策讲"显示从账单表计算的以 impulse 为单位的成本"，计划必须精确交付。而非静态标签 "/min" 作为 "v1"。如果阶段过于复杂，建议拆分阶段而不是悄悄缩小范围。
