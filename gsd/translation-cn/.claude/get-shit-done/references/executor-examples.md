# Executor 扩展示例

> gsd-executor agent 的参考文件。通过 `@` 引用按需加载。
> 对于低于 200K 的上下文窗口，此内容从 agent prompt 中剥离，在此按需加载。

## 偏差规则示例

### 规则 1——自动修复 bug

**规则 1 触发示例：**
- 返回错误数据的错误查询
- 条件判断中的逻辑错误
- 类型错误和类型不匹配
- 空指针异常 / undefined 访问
- 损坏的验证（接受无效输入）
- 安全漏洞（XSS、SQL 注入）
- 异步代码中的竞态条件
- 未清理资源导致的内存泄漏

### 规则 2——自动添加缺失的关键功能

**规则 2 触发示例：**
- 缺失错误处理（未处理的 promise 拒绝，I/O 无 try/catch）
- 面向用户的端点无输入验证
- 属性访问前缺少空检查
- 受保护路由无认证
- 缺失授权检查（用户可以访问其他用户的数据）
- 无 CSRF/CORS 配置
- 公共端点无速率限制
- 频繁查询列缺少数据库索引
- 无错误日志（失败被默默吞没）

### 规则 3——自动修复阻塞问题

**规则 3 触发示例：**
- package.json 中缺少依赖
- 阻止编译的错误类型
- 破损的导入（错误路径、错误导出名）
- 运行时需要的环境变量缺失
- 数据库连接错误（错误 URL、缺少凭证）
- 构建配置错误（错误入口点、缺少加载器）
- 引用文件缺失（导入指向不存在的模块）
- 阻止模块加载的循环依赖

### 规则 4——询问架构变更

**规则 4 触发示例：**
- 新建数据库表（不仅是添加列）
- 重大模式变更（重命名表、更改关系）
- 新建服务层（添加队列、缓存或消息总线）
- 切换库/框架（例如，用 Fastify 替换 Express）
- 更改认证方式（从会话切换到 JWT）
- 新基础设施（添加 Redis、S3 等）
- 破坏性 API 变更（删除或重命名端点）

## 边界情况决策指南

| 场景 | 规则 | 理由 |
|----------|------|-----------|
| 输入缺少验证 | 规则 2 | 安全要求 |
| 空输入导致崩溃 | 规则 1 | Bug——不正确行为 |
| 需要新建数据库表 | 规则 4 | 架构决策 |
| 需要向现有表添加列 | 规则 1 或 2 | 取决于上下文 |
| 预先存在的 lint 警告 | 超出范围 | 不是当前任务引起的 |
| 无关的测试失败 | 超出范围 | 不是当前任务引起的 |

**决策启发式：** "这会影响正确性、安全性或完成当前任务的能力吗？"
- 是 → 规则 1-3（自动修复）
- 可能 → 规则 4（询问用户）
- 否 → 超出范围（记录到 deferred-items.md）

## 检查点示例

### 好的检查点位置

```xml
<!-- 自动化一切，最后验证 -->
<task type="auto">创建数据库模式</task>
<task type="auto">创建 API 端点</task>
<task type="auto">创建 UI 组件</task>
<task type="checkpoint:human-verify">
  <what-built>完整认证流程（模式 + API + UI）</what-built>
  <how-to-verify>
    1. 访问 http://localhost:3000/register
    2. 使用 test@example.com 创建账户
    3. 用这些凭证登录
    4. 验证仪表盘加载并显示用户名
  </how-to-verify>
</task>
```

### 坏的检查点位置

```xml
<!-- 检查点太多——导致验证疲劳 -->
<task type="auto">创建模式</task>
<task type="checkpoint:human-verify">检查模式</task>
<task type="auto">创建 API</task>
<task type="checkpoint:human-verify">检查 API</task>
<task type="auto">创建 UI</task>
<task type="checkpoint:human-verify">检查 UI</task>
```

### 认证阻碍处理

当 `type="auto"` 执行期间出现认证错误时：
1. 将其识别为认证阻碍（非 bug）——指标："Not authenticated"、"401"、"403"、"Please run X login"
2. 停止当前任务
3. 返回包含精确认证步骤的 `checkpoint:human-action`
4. 在 SUMMARY.md 中，将认证阻碍记录为正常流程，而非偏差
