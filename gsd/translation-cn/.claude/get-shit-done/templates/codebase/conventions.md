# 编码约定模板

`.planning/codebase/CONVENTIONS.md` 的模板——记录编码风格和模式。

**目的：** 记录本代码库中代码的编写方式。为 Claude 匹配现有风格提供规范性指南。

---

## 文件模板

```markdown
# 编码约定

**分析日期：** [YYYY-MM-DD]

## 命名模式

**文件：**
- [模式：例如"所有文件使用 kebab-case"]
- [测试文件：例如"*.test.ts 与源文件并列"]
- [组件：例如"React 组件使用 PascalCase.tsx"]

**函数：**
- [模式：例如"所有函数使用 camelCase"]
- [异步：例如"异步函数无特殊前缀"]
- [处理函数：例如"事件处理函数使用 handleEventName"]

**变量：**
- [模式：例如"变量使用 camelCase"]
- [常量：例如"常量使用 UPPER_SNAKE_CASE"]
- [私有：例如"_前缀用于私有成员" 或 "无前缀"]

**类型：**
- [接口：例如"PascalCase，无 I 前缀"]
- [类型：例如"类型别名使用 PascalCase"]
- [枚举：例如"枚举名使用 PascalCase，值使用 UPPER_CASE"]

## 代码风格

**格式化：**
- [工具：例如"Prettier，配置在 .prettierrc"]
- [行长：例如"最大 100 个字符"]
- [引号：例如"字符串使用单引号"]
- [分号：例如"必须" 或 "省略"]

**Linting：**
- [工具：例如"ESLint，配置在 eslint.config.js"]
- [规则：例如"继承 airbnb-base，生产环境禁止 console"]
- [运行：例如"npm run lint"]

## 导入组织

**顺序：**
1. [例如"外部包（react、express 等）"]
2. [例如"内部模块（@/lib、@/components）"]
3. [例如"相对导入（.、..）"]
4. [例如"类型导入（import type {}）"]

**分组：**
- [空行：例如"各组之间留空行"]
- [排序：例如"每组内按字母顺序"]

**路径别名：**
- [使用的别名：例如"@/ 映射 src/，@components/ 映射 src/components/"]

## 错误处理

**模式：**
- [策略：例如"抛出错误，在边界捕获"]
- [自定义错误：例如"扩展 Error 类，命名 *Error"]
- [异步：例如"使用 try/catch，不使用 .catch() 链"]

**错误类型：**
- [何时抛出：例如"无效输入、缺失依赖"]
- [何时返回：例如"预期失败返回 Result<T, E>"]
- [日志：例如"抛前记录错误及上下文"]

## 日志

**框架：**
- [工具：例如"console.log、pino、winston"]
- [级别：例如"debug、info、warn、error"]

**模式：**
- [格式：例如"结构化日志，带上下文对象"]
- [时机：例如"记录状态转换、外部调用"]
- [位置：例如"在服务边界记录，不在工具函数中"]

## 注释

**何时添加注释：**
- [例如"解释为什么，而非做了什么"]
- [例如"记录业务逻辑、算法、边界情况"]
- [例如"避免显而易见的注释，如 // increment counter"]

**JSDoc/TSDoc：**
- [用法：例如"公共 API 必须，内部可选"]
- [格式：例如"使用 @param、@returns、@throws 标签"]

**TODO 注释：**
- [模式：例如"// TODO(username)：描述"]
- [跟踪：例如"链接到 issue 编号（如有）"]

## 函数设计

**大小：**
- [例如"保持在 50 行以下，提取辅助函数"]

**参数：**
- [例如"最多 3 个参数，超过则使用对象"]
- [例如"在参数列表中解构对象"]

**返回值：**
- [例如"显式返回，不隐式返回 undefined"]
- [例如"守卫子句使用早期返回"]

## 模块设计

**导出：**
- [例如"优先命名导出，React 组件使用默认导出"]
- [例如"通过 index.ts 导出公共 API"]

**桶文件：**
- [例如"使用 index.ts 重新导出公共 API"]
- [例如"避免循环依赖"]

---

*约定分析：[日期]*
*当模式发生变化时更新*
```

<good_examples>
```markdown
# 编码约定

**分析日期：** 2025-01-20

## 命名模式

**文件：**
- 所有文件使用 kebab-case（command-handler.ts、user-service.ts）
- *.test.ts 与源文件并列
- index.ts 用于桶导出

**函数：**
- 所有函数使用 camelCase
- 异步函数无特殊前缀
- 事件处理函数使用 handleEventName（handleClick、handleSubmit）

**变量：**
- 变量使用 camelCase
- 常量使用 UPPER_SNAKE_CASE（MAX_RETRIES、API_BASE_URL）
- 无下划线前缀（TS 中无私有标记）

**类型：**
- 接口使用 PascalCase，无 I 前缀（User，而非 IUser）
- 类型别名使用 PascalCase（UserConfig、ResponseData）
- 枚举名使用 PascalCase，值使用 UPPER_CASE（Status.PENDING）

## 代码风格

**格式化：**
- Prettier，配置在 .prettierrc
- 100 字符行长
- 字符串使用单引号
- 分号必须
- 2 空格缩进

**Linting：**
- ESLint，配置在 eslint.config.js
- 继承 @typescript-eslint/recommended
- 生产代码中禁止 console.log（使用 logger）
- 运行：npm run lint

## 导入组织

**顺序：**
1. 外部包（react、express、commander）
2. 内部模块（@/lib、@/services）
3. 相对导入（./utils、../types）
4. 类型导入（import type { User }）

**分组：**
- 各组之间留空行
- 每组内按字母顺序
- 类型导入在每组内排最后

**路径别名：**
- @/ 映射 src/
- 无其他别名定义

## 错误处理

**模式：**
- 抛出错误，在边界捕获（路由处理函数、main 函数）
- 扩展 Error 类用于自定义错误（ValidationError、NotFoundError）
- 异步函数使用 try/catch，不使用 .catch() 链

**错误类型：**
- 无效输入、缺失依赖、不可变违规时抛出
- 抛前记录错误及上下文：logger.error({ err, userId }，'Failed to process')
- 在错误消息中包含 cause：new Error('Failed to X'，{ cause: originalError })

## 日志

**框架：**
- 从 lib/logger.ts 导出的 pino logger 实例
- 级别：debug、info、warn、error（无 trace）

**模式：**
- 结构化日志，带上下文：logger.info({ userId, action }，'User action')
- 在服务边界记录，不在工具函数中
- 记录状态转换、外部 API 调用、错误
- 提交的代码中无 console.log

## 注释

**何时添加注释：**
- 解释为什么，而非做了什么：// Retry 3 times because API has transient failures
- 记录业务规则：// Users must verify email within 24 hours
- 解释非显而易见的算法或临时方案
- 避免显而易见的注释：// set count to 0

**JSDoc/TSDoc：**
- 公共 API 函数必须
- 内部函数如果签名能自解释则可选
- 使用 @param、@returns、@throws 标签

**TODO 注释：**
- 格式：// TODO：描述（无用户名，使用 git blame）
- 如有 issue 则链接：// TODO：Fix race condition (issue #123)

## 函数设计

**大小：**
- 保持在 50 行以下
- 提取辅助函数处理复杂逻辑
- 每个函数一个抽象层级

**参数：**
- 最多 3 个参数
- 4+ 参数使用 options 对象：function create(options: CreateOptions)
- 在参数列表中解构：function process({ id, name }: ProcessParams)

**返回值：**
- 显式返回语句
- 守卫子句使用早期返回
- 预期失败使用 Result<T, E> 类型

## 模块设计

**导出：**
- 优先命名导出
- 仅 React 组件使用默认导出
- 通过 index.ts 桶文件导出公共 API

**桶文件：**
- index.ts 重新导出公共 API
- 保持内部辅助函数为私有（不从 index 导出）
- 避免循环依赖（如需要可从具体文件导入）

---

*约定分析：2025-01-20*
*当模式发生变化时更新*
```
</good_examples>

<guidelines>
**CONVENTIONS.md 应包含的内容：**
- 代码库中观察到的命名模式
- 格式化规则（Prettier 配置、linting 规则）
- 导入组织模式
- 错误处理策略
- 日志方法
- 注释约定
- 函数和模块设计模式

**不应包含的内容：**
- 架构决策（那是 ARCHITECTURE.md 的事）
- 技术选择（那是 STACK.md 的事）
- 测试模式（那是 TESTING.md 的事）
- 文件组织（那是 STRUCTURE.md 的事）

**填充此模板时：**
- 检查 .prettierrc、.eslintrc 或类似的配置文件
- 检查 5-10 个代表性源文件的模式
- 寻找一致性：如果 80%+ 遵循某个模式，记录下来
- 要规范："使用 X" 而非"有时使用 Y"
- 注意偏差："遗留代码使用 Y，新代码应使用 X"
- 总篇幅控制在约 150 行以内

**在以下场景中对阶段规划有用：**
- 编写新代码（匹配现有风格）
- 添加功能（遵循命名模式）
- 重构（应用一致的约定）
- 代码审查（对照已记录的模式检查）
- 引导（理解风格预期）

**分析方法：**
- 扫描 src/ 目录寻找文件命名模式
- 检查 package.json scripts 中的 lint/format 命令
- 阅读 5-10 个文件识别函数命名、错误处理
- 寻找配置文件（.prettierrc、eslint.config.js）
- 注意导入、注释、函数签名中的模式
</guidelines>
