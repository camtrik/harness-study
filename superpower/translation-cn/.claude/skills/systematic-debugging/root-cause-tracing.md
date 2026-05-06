# 根本原因追踪

## 概述

缺陷常常在调用栈深处显现（git init 在错误目录、文件创建在错误位置、数据库以错误路径打开）。你的直觉是在错误出现的地方修复，但这是在治标不治本。

**核心原则：** 沿着调用链反向追踪，直到找到最初的触发源，然后在源头修复。

## 何时使用

```dot
digraph when_to_use {
    "缺陷是否在调用栈深处出现？" [shape=diamond];
    "能否反向追踪？" [shape=diamond];
    "在症状点修复" [shape=box];
    "追踪到最初触发源" [shape=box];
    "更好：同时添加纵深防御" [shape=box];

    "缺陷是否在调用栈深处出现？" -> "能否反向追踪？" [label="是"];
    "能否反向追踪？" -> "追踪到最初触发源" [label="是"];
    "能否反向追踪？" -> "在症状点修复" [label="否 - 死胡同"];
    "追踪到最初触发源" -> "更好：同时添加纵深防御";
}
```

**适用场景：**
- 错误在执行深处发生（而非入口点）
- 堆栈追踪显示长调用链
- 不清楚无效数据从哪里产生
- 需要找出是哪个测试/代码触发了问题

## 追踪过程

### 1. 观察症状
```
Error: git init 在 ~/project/packages/core 中失败
```

### 2. 找到直接原因
**什么代码直接导致了这个问题？**
```typescript
await execFileAsync('git', ['init'], { cwd: projectDir });
```

### 3. 问：是什么调用了它？
```typescript
WorktreeManager.createSessionWorktree(projectDir, sessionId)
  → 被 Session.initializeWorkspace() 调用
  → 被 Session.create() 调用
  → 被测试在 Project.create() 处调用
```

### 4. 继续向上追踪
**传入了什么值？**
- `projectDir = ''`（空字符串！）
- 空字符串作为 `cwd` 解析为 `process.cwd()`
- 那是源代码目录！

### 5. 找到最初触发源
**空字符串从哪里来？**
```typescript
const context = setupCoreTest(); // 返回 { tempDir: '' }
Project.create('name', context.tempDir); // 在 beforeEach 之前访问！
```

## 添加堆栈追踪

当你无法手动追踪时，添加检测代码：

```typescript
// 在有问题的操作之前
async function gitInit(directory: string) {
  const stack = new Error().stack;
  console.error('DEBUG git init:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });

  await execFileAsync('git', ['init'], { cwd: directory });
}
```

**关键：** 在测试中使用 `console.error()`（而不是 logger——可能不会显示）

**运行并捕获：**
```bash
npm test 2>&1 | grep 'DEBUG git init'
```

**分析堆栈追踪：**
- 查找测试文件名
- 找到触发调用的行号
- 识别模式（同一测试？同一参数？）

## 找到哪个测试造成了污染

如果某个东西在测试期间出现但不知道是哪个测试导致的：

使用本目录中的二分查找脚本 `find-polluter.sh`：

```bash
./find-polluter.sh '.git' 'src/**/*.test.ts'
```

逐一运行测试，在第一个污染源处停止。详见脚本使用说明。

## 真实案例：空的 projectDir

**症状：** `.git` 被创建在 `packages/core/`（源代码中）

**追踪链：**
1. `git init` 在 `process.cwd()` 中执行 ← 空的 cwd 参数
2. WorktreeManager 被传入了空的 projectDir
3. Session.create() 传入了空字符串
4. 测试在 beforeEach 之前访问了 `context.tempDir`
5. setupCoreTest() 初始返回 `{ tempDir: '' }`

**根本原因：** 顶层变量初始化时访问了空值

**修复：** 将 tempDir 改为 getter，在 beforeEach 之前访问时抛出错误

**同时添加了纵深防御：**
- 第 1 层：Project.create() 验证目录
- 第 2 层：WorkspaceManager 验证非空
- 第 3 层：NODE_ENV 守卫拒绝在 tmpdir 之外执行 git init
- 第 4 层：git init 前记录堆栈追踪

## 核心原则

```dot
digraph principle {
    "找到直接原因" [shape=ellipse];
    "能否再追踪一层？" [shape=diamond];
    "反向追踪" [shape=box];
    "这是源头吗？" [shape=diamond];
    "在源头修复" [shape=box];
    "在每一层添加验证" [shape=box];
    "缺陷不可能发生" [shape=doublecircle];
    "绝不只修复症状" [shape=octagon, style=filled, fillcolor=red, fontcolor=white];

    "找到直接原因" -> "能否再追踪一层？";
    "能否再追踪一层？" -> "反向追踪" [label="是"];
    "能否再追踪一层？" -> "绝不只修复症状" [label="否"];
    "反向追踪" -> "这是源头吗？";
    "这是源头吗？" -> "反向追踪" [label="否 - 继续"];
    "这是源头吗？" -> "在源头修复" [label="是"];
    "在源头修复" -> "在每一层添加验证";
    "在每一层添加验证" -> "缺陷不可能发生";
}
```

**绝不只修复错误出现的地方。** 反向追踪找到最初的触发源。

## 堆栈追踪技巧

**在测试中：** 使用 `console.error()` 而非 logger——logger 可能被抑制
**在操作之前：** 在危险操作之前记录日志，而非失败之后
**包含上下文：** 目录、cwd、环境变量、时间戳
**捕获堆栈：** `new Error().stack` 显示完整调用链

## 实际效果

来自调试会话（2025-10-03）：
- 通过 5 层追踪找到根本原因
- 在源头修复（getter 验证）
- 添加了 4 层防线
- 1847 个测试通过，零污染
