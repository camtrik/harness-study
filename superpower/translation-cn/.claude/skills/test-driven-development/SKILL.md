---
name: test-driven-development
description: 在实现任何功能或修复 bug 时使用，在编写实现代码之前
---

# 测试驱动开发 (TDD)

## 概述

先编写测试。看着它失败。编写最少的代码来通过测试。

**核心原则：**如果你没有看着测试失败，你就不知道它是否测试了正确的内容。

**违反规则的字面意思就是违反规则的精神。**

## 何时使用

**始终：**
- 新功能
- Bug 修复
- 重构
- 行为变更

**例外情况（请询问你的 人类搭档）：**
- 一次性原型
- 生成的代码
- 配置文件

想"这次就跳过 TDD"？停下。这是自我合理化。

## 铁律

```
没有失败的测试就不能编写生产代码
```

在测试之前编写代码？删除它。重新开始。

**没有例外：**
- 不要将其保留为"参考"
- 不要在编写测试时"调整"它
- 不要看它
- 删除意味着删除

从头开始实现。就这样。

## 红-绿-重构

```dot
digraph tdd_cycle {
    rankdir=LR;
    red [label="RED\n编写失败的测试", shape=box, style=filled, fillcolor="#ffcccc"];
    verify_red [label="验证失败\n正确", shape=diamond];
    green [label="GREEN\n最少的代码", shape=box, style=filled, fillcolor="#ccffcc"];
    verify_green [label="验证通过\n全部绿色", shape=diamond];
    refactor [label="REFACTOR\n清理", shape=box, style=filled, fillcolor="#ccccff"];
    next [label="下一个", shape=ellipse];

    red -> verify_red;
    verify_red -> green [label="是"];
    verify_red -> red [label="错误\n的失败"];
    green -> verify_green;
    verify_green -> refactor [label="是"];
    verify_green -> green [label="否"];
    refactor -> verify_green [label="保持\n绿色"];
    verify_green -> next;
    next -> red;
}
```

### RED - 编写失败的测试

编写一个最小的测试来展示应该发生什么。

<推荐>
```typescript
test('重试失败的操作 3 次', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };

  const result = await retryOperation(operation);

  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```
名称清晰，测试真实行为，只做一件事
</推荐>

<不推荐>
```typescript
test('重试有效', async () => {
  const mock = jest.fn()
    .mockRejectedValueOnce(new Error())
    .mockRejectedValueOnce(new Error())
    .mockResolvedValueOnce('success');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(3);
});
```
名称模糊，测试 mock 而不是代码
</不推荐>

**要求：**
- 一个行为
- 名称清晰
- 真实代码（除非不可避免，否则不使用 mock）

### 验证 RED - 看着它失败

**必须。永远不要跳过。**

```bash
npm test path/to/test.test.ts
```

确认：
- 测试失败（不是错误）
- 失败消息符合预期
- 因功能缺失而失败（不是拼写错误）

**测试通过了？**你正在测试现有行为。修复测试。

**测试出错了？**修复错误，重新运行直到它正确失败。

### GREEN - 最少的代码

编写最简单的代码来通过测试。

<推荐>
```typescript
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === 2) throw e;
    }
  }
  throw new Error('unreachable');
}
```
刚好通过测试
</推荐>

<不推荐>
```typescript
async function retryOperation<T>(
  fn: () => Promise<T>,
  options?: {
    maxRetries?: number;
    backoff?: 'linear' | 'exponential';
    onRetry?: (attempt: number) => void;
  }
): Promise<T> {
  // YAGNI
}
```
过度工程化
</不推荐>

不要添加功能、重构其他代码或在测试之外"改进"。

### 验证 GREEN - 看着它通过

**必须。**

```bash
npm test path/to/test.test.ts
```

确认：
- 测试通过
- 其他测试仍然通过
- 输出干净（没有错误、警告）

**测试失败？**修复代码，而不是测试。

**其他测试失败？**立即修复。

### REFACTOR - 清理

仅在绿色之后：
- 删除重复
- 改进命名
- 提取辅助函数

保持测试绿色。不要添加行为。

### 重复

为下一个功能编写下一个失败的测试。

## 好的测试

| 质量 | 推荐 | 不推荐 |
|---------|------|-----|
| **最小化** | 一件事。名称中有"and"？拆分它。 | `test('验证邮箱和域名以及空白字符')` |
| **清晰** | 名称描述行为 | `test('test1')` |
| **显示意图** | 展示所需的 API | 掩盖代码应该做什么 |

## 为什么顺序很重要

**"我会在之后编写测试来验证它有效"**

在代码之后编写的测试立即通过。立即通过证明不了什么：
- 可能测试了错误的东西
- 可能测试了实现，而不是行为
- 可能遗漏了你忘记的边界情况
- 你从未看到它捕获 bug

测试优先迫使你看到测试失败，证明它确实测试了某些东西。

**"我已经手动测试了所有边界情况"**

手动测试是临时的。你以为你测试了一切，但：
- 没有测试记录
- 代码更改时无法重新运行
- 在压力下容易忘记情况
- "我尝试时它有效" ≠ 全面

自动化测试是系统性的。它们每次都以相同的方式运行。

**"删除 X 小时的工作是浪费"**

沉没成本谬误。时间已经过去了。你现在有两个选择：
- 删除并使用 TDD 重写（X 更多小时，高信心）
- 保留它并在之后添加测试（30 分钟，低信心，可能有 bug）

"浪费"是保留你无法信任的代码。没有真正测试的工作代码是技术债务。

**"TDD 是教条的，务实意味着要适应"**

TDD 是务实的：
- 在提交之前发现 bug（比之后调试更快）
- 防止回归（测试立即捕获中断）
- 记录行为（测试展示如何使用代码）
- 启用重构（自由更改，测试捕获中断）

"务实"的捷径 = 在生产环境中调试 = 更慢。

**"测试后能达到相同的目标 - 这是精神而非仪式"**

不。测试后回答"这做什么？"。测试优先回答"这应该做什么？"。

测试后被你的实现所偏见。你测试你构建的东西，而不是需要的东西。你验证记得的边界情况，而不是发现的边界情况。

测试优先在实现之前强制发现边界情况。测试后验证你是否记得所有事情（你不会）。

30 分钟的测试后 ≠ TDD。你获得了覆盖率，失去了测试有效的证明。

## 常见的自我合理化

| 借口 | 现实 |
|--------|---------|
| "太简单了无法测试" | 简单的代码会出错。测试需要 30 秒。 |
| "我会在之后测试" | 测试立即通过证明不了什么。 |
| "测试后能达到相同的目标" | 测试后 = "这做什么？"测试优先 = "这应该做什么？" |
| "已经手动测试" | 临时的 ≠ 系统性的。没有记录，无法重新运行。 |
| "删除 X 小时是浪费的" | 沉没成本谬误。保留未经验证的代码是技术债务。 |
| "保留作为参考，先编写测试" | 你会调整它。这是测试后。删除意味着删除。 |
| "需要先探索" | 可以。丢弃探索，从 TDD 开始。 |
| "测试很难 = 设计不清楚" | 听从测试。难以测试 = 难以使用。 |
| "TDD 会拖慢我的速度" | TDD 比调试更快。务实 = 测试优先。 |
| "手动测试更快" | 手动不证明边界情况。你会重新测试每次更改。 |
| "现有代码没有测试" | 你正在改进它。为现有代码添加测试。 |

## 危险信号 - 停下并重新开始

- 测试之前的代码
- 实现后的测试
- 测试立即通过
- 无法解释为什么测试失败
- "稍后"添加测试
- 合理化"就这一次"
- "我已经手动测试了它"
- "测试后能达到相同的目的"
- "这是精神而不是仪式"
- "保留作为参考"或"调整现有代码"
- "已经花了 X 小时，删除是浪费的"
- "TDD 是教条的，我是务实的"
- "这是不同的，因为..."

**所有这些都意味着：删除代码。使用 TDD 重新开始。**

## 示例：Bug 修复

**Bug：**接受空邮箱

**RED**
```typescript
test('拒绝空邮箱', async () => {
  const result = await submitForm({ email: '' });
  expect(result.error).toBe('Email required');
});
```

**验证 RED**
```bash
$ npm test
FAIL: expected 'Email required', got undefined
```

**GREEN**
```typescript
function submitForm(data: FormData) {
  if (!data.email?.trim()) {
    return { error: 'Email required' };
  }
  // ...
}
```

**验证 GREEN**
```bash
$ npm test
PASS
```

**REFACTOR**
如果需要，为多个字段提取验证。

## 验证清单

在标记工作完成之前：

- [ ] 每个新函数/方法都有测试
- [ ] 在实现之前看着每个测试失败
- [ ] 每个测试都因预期原因而失败（功能缺失，不是拼写错误）
- [ ] 编写了最少的代码来通过每个测试
- [ ] 所有测试都通过
- [ ] 输出干净（没有错误、警告）
- [ ] 测试使用真实代码（仅当不可避免时才使用 mock）
- [ ] 覆盖了边界情况和错误

无法检查所有框？你跳过了 TDD。重新开始。

## 遇到困难时

| 问题 | 解决方案 |
|---------|----------|
| 不知道如何测试 | 编写期望的 API。先编写断言。询问你的 人类搭档。 |
| 测试太复杂 | 设计太复杂。简化接口。 |
| 必须模拟所有内容 | 代码耦合太紧密。使用依赖注入。 |
| 测试设置太大 | 提取辅助函数。仍然复杂？简化设计。 |

## 调试集成

发现了 bug？编写重现它的失败测试。遵循 TDD 周期。测试证明修复并防止回归。

永远不要在没有测试的情况下修复 bug。

## 测试反模式

当添加 mock 或测试工具时，阅读 @testing-anti-patterns.md 以避免常见陷阱：
- 测试 mock 行为而不是真实行为
- 向生产类添加仅测试方法
- 在不理解依赖项的情况下进行 mock

## 最终规则

```
生产代码 → 测试存在并且先失败
否则 → 不是 TDD
```

没有你的 人类搭档许可，没有例外。
