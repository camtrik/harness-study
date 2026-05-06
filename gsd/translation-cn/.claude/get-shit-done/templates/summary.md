# 摘要模板

`.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md` 的模板——阶段完成文档。

---

## 文件模板

```markdown
---
phase: XX-name
plan: YY
subsystem: [主要分类：auth、payments、ui、api、database、infra、testing 等]
tags: [可搜索的技术标签：jwt、stripe、react、postgres、prisma]

# 依赖图
requires:
  - phase: [此阶段依赖的前置阶段]
    provides: [该阶段构建了什么供此阶段使用]
provides:
  - [此阶段构建/交付的要点列表]
affects: [需要此上下文的阶段名称或关键词列表]

# 技术跟踪
tech-stack:
  added: [此阶段新增的库/工具]
  patterns: [建立的架构/代码模式]

key-files:
  created: [创建的重要文件]
  modified: [修改的重要文件]

key-decisions:
  - "决策 1"
  - "决策 2"

patterns-established:
  - "模式 1：描述"
  - "模式 2：描述"

requirements-completed: []  # 必需——从此计划的 `requirements` frontmatter 字段复制所有需求 ID。

# 指标
duration: Xmin
completed: YYYY-MM-DD
---

# 阶段 [X]：[名称] 摘要

**[描述成果的实质性一句话——不是"阶段完成"或"实现已完成"]**

## 性能

- **耗时：** [时间]（例如 23 min，1h 15m）
- **开始：** [ISO 时间戳]
- **完成：** [ISO 时间戳]
- **任务：** [完成数量]
- **文件修改：** [数量]

## 成就
- [最重要的成果]
- [第二个关键成就]
- [第三个（如适用）]

## 任务提交

每个任务原子提交：

1. **任务 1：[任务名称]** - `abc123f` (feat/fix/test/refactor)
2. **任务 2：[任务名称]** - `def456g` (feat/fix/test/refactor)
3. **任务 3：[任务名称]** - `hij789k` (feat/fix/test/refactor)

**计划元数据：** `lmn012o` (docs: complete plan)

_注意：TDD 任务可能有多个提交（test → feat → refactor）_

## 创建/修改的文件
- `path/to/file.ts` - 功能描述
- `path/to/another.ts` - 功能描述

## 做出的决策
[关键决策及简要理由，或"无——按计划执行"]

## 与计划的偏差

[如果没有偏差："无——严格按照计划执行"]

[如果有偏差：]

### 自动修复的问题

**1. [规则 X - 分类] 简要描述**
- **发现期间：** 任务 [N]（[任务名称]）
- **问题：** [哪里不对]
- **修复：** [做了什么]
- **修改的文件：** [文件路径]
- **验证：** [如何验证]
- **提交于：** [hash]（任务提交的一部分）

[... 对每个自动修复重复 ...]

---

**总偏差数：** [N] 个自动修复（[按规则细分]）
**对计划的影响：** [简要评估——例如"所有自动修复对正确性/安全性都是必需的。无范围蔓延。"]

## 遇到的问题
[问题及如何解决，或"无"]

[注意："与计划的偏差"记录通过偏差规则自动处理的非计划工作。"遇到的问题"记录计划工作中需要问题解决的困难。]

## 需要用户设置

[如果生成了 USER-SETUP.md：]
**外部服务需要手动配置。** 参见 [{phase}-USER-SETUP.md](./{phase}-USER-SETUP.md)：
- 要添加的环境变量
- 仪表盘配置步骤
- 验证命令

[如果没有 USER-SETUP.md：]
无——不需要外部服务配置。

## 下一阶段准备情况
[为下一阶段准备就绪的内容]
[任何阻塞项或关注点]

---
*阶段：XX-name*
*完成日期：[日期]*
```

<frontmatter_guidance>
**目的：** 通过依赖图启用自动上下文组装。Frontmatter 使摘要元数据机器可读，因此 plan-phase 可以快速扫描所有摘要并根据依赖关系选择相关的。

**快速扫描：** Frontmatter 在前约 25 行内，可以廉价地跨所有摘要扫描而无需阅读完整内容。

**依赖图：** `requires`/`provides`/`affects` 在阶段之间创建显式链接，实现上下文选择的传递闭包。

**子系统：** 主要分类（auth、payments、ui、api、database、infra、testing）用于检测相关阶段。

**标签：** 可搜索的技术关键词（库、框架、工具）用于技术栈感知。

**关键文件：** PLAN.md 中 @context 引用的重要文件。

**模式：** 未来阶段应保持的已建立约定。

**填充：** Frontmatter 在 execute-plan.md 中的摘要创建期间填充。逐字段指导参见 `<step name="create_summary">`。
</frontmatter_guidance>

<one_liner_rules>
一句话必须是实质性的：

**好的：**
- "使用 jose 库的带刷新旋转的 JWT 认证"
- "含 User、Session 和 Product 模型的 Prisma schema"
- "通过 Server-Sent Events 的实时指标仪表盘"

**不好的：**
- "阶段完成"
- "认证已实现"
- "基础已完成"
- "所有任务已完成"

一句话应该告诉别人实际发布了什么。
</one_liner_rules>

<example>
```markdown
# 阶段 1：基础摘要

**使用 jose 库的带刷新旋转的 JWT 认证、Prisma User 模型和受保护的 API 中间件**

## 性能

- **耗时：** 28 min
- **开始：** 2025-01-15T14:22:10Z
- **完成：** 2025-01-15T14:50:33Z
- **任务：** 5
- **文件修改：** 8

## 成就
- 含邮箱/密码认证的 User 模型
- 使用 httpOnly JWT cookies 的登录/登出端点
- 检查 token 有效性的受保护路由中间件
- 每次请求刷新 token 旋转

## 创建/修改的文件
- `prisma/schema.prisma` - User 和 Session 模型
- `src/app/api/auth/login/route.ts` - 登录端点
- `src/app/api/auth/logout/route.ts` - 登出端点
- `src/middleware.ts` - 受保护路由检查
- `src/lib/auth.ts` - 使用 jose 的 JWT 辅助函数

## 做出的决策
- 使用 jose 代替 jsonwebtoken（ESM-native，Edge 兼容）
- 15 分钟访问 token 配 7 天刷新 token
- 将刷新 token 存储在数据库中以支持撤销能力

## 与计划的偏差

### 自动修复的问题

**1. [规则 2 - 缺失关键项] 添加了 bcrypt 密码哈希**
- **发现期间：** 任务 2（登录端点实现）
- **问题：** 计划未指定密码哈希——存储明文将是关键安全缺陷
- **修复：** 在注册时添加 bcrypt 哈希，在登录时比较，salt rounds 10
- **修改的文件：** src/app/api/auth/login/route.ts, src/lib/auth.ts
- **验证：** 密码哈希测试通过，绝不存储明文
- **提交于：** abc123f（任务 2 提交）

**2. [规则 3 - 阻塞项] 安装了缺失的 jose 依赖**
- **发现期间：** 任务 4（JWT token 生成）
- **问题：** jose 包不在 package.json 中，导入失败
- **修复：** 运行 `npm install jose`
- **修改的文件：** package.json, package-lock.json
- **验证：** 导入成功，构建通过
- **提交于：** def456g（任务 4 提交）

---

**总偏差数：** 2 个自动修复（1 个缺失关键项，1 个阻塞项）
**对计划的影响：** 两项自动修复对安全性和功能性都是必需的。无范围蔓延。

## 遇到的问题
- jsonwebtoken CommonJS 导入在 Edge 运行时失败——切换到 jose（计划的库变更，按预期工作）

## 下一阶段准备情况
- 认证基础完成，可开始功能开发
- 公开发布前需要用户注册端点

---
*阶段：01-foundation*
*完成日期：2025-01-15*
```
</example>

<guidelines>
**Frontmatter：** 强制——完成所有字段。为未来规划启用自动上下文组装。

**一句话：** 必须是实质性的。"使用 jose 库的带刷新旋转的 JWT 认证"而非"认证已实现"。

**决策段：**
- 执行期间做出的关键决策及理由
- 提取到 STATE.md 累积上下文
- 如果没有偏差使用"无——按计划执行"

**创建后：** STATE.md 更新位置、决策、问题。
</guidelines>
