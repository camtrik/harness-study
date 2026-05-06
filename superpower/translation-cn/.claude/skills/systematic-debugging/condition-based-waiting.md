# 基于条件的等待

## 概述

不稳定的测试常常使用任意延迟来猜测时机。这会产生竞态条件——测试在快速机器上通过，但在负载下或在 CI 中失败。

**核心原则：** 等待你真正关心的条件，而不是猜测需要多长时间。

## 何时使用

```dot
digraph when_to_use {
    "测试是否使用了 setTimeout/sleep？" [shape=diamond];
    "是否在测试时间行为？" [shape=diamond];
    "记录为什么需要超时" [shape=box];
    "使用基于条件的等待" [shape=box];

    "测试是否使用了 setTimeout/sleep？" -> "是否在测试时间行为？" [label="是"];
    "是否在测试时间行为？" -> "记录为什么需要超时" [label="是"];
    "是否在测试时间行为？" -> "使用基于条件的等待" [label="否"];
}
```

**适用场景：**
- 测试中使用了任意延迟（`setTimeout`、`sleep`、`time.sleep()`）
- 测试不稳定（有时通过，负载下失败）
- 测试在并行运行时超时
- 等待异步操作完成

**不适用场景：**
- 测试实际的时间行为（防抖、节流间隔）
- 如果使用任意超时，务必记录原因

## 核心模式

```typescript
// ❌ 之前：猜测时机
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ 之后：等待条件满足
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## 快速参考

| 场景 | 模式 |
|------|------|
| 等待事件 | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| 等待状态 | `waitFor(() => machine.state === 'ready')` |
| 等待数量 | `waitFor(() => items.length >= 5)` |
| 等待文件 | `waitFor(() => fs.existsSync(path))` |
| 复杂条件 | `waitFor(() => obj.ready && obj.value > 10)` |

## 实现

通用轮询函数：
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`等待 ${description} 超时，已等待 ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // 每 10ms 轮询一次
  }
}
```

参见本目录中的 `condition-based-waiting-example.ts`，其中包含来自实际调试会话的完整实现和特定领域的辅助函数（`waitForEvent`、`waitForEventCount`、`waitForEventMatch`）。

## 常见错误

**❌ 轮询过快：** `setTimeout(check, 1)`——浪费 CPU
**✅ 修复：** 每 10ms 轮询一次

**❌ 没有超时：** 如果条件永远不满足，会无限循环
**✅ 修复：** 始终包含带清晰错误信息的超时

**❌ 使用过期数据：** 在循环前缓存状态
**✅ 修复：** 在循环内调用 getter 以获取最新数据

## 何时任意超时是合理的

```typescript
// 工具每 100ms 执行一次——需要等待 2 次来验证部分输出
await waitForEvent(manager, 'TOOL_STARTED'); // 首先：等待条件
await new Promise(r => setTimeout(r, 200));   // 然后：等待时间行为
// 200ms = 间隔 100ms 的 2 次执行——已记录和说明原因
```

**要求：**
1. 首先等待触发条件
2. 基于已知的时间间隔（而非猜测）
3. 用注释解释原因

## 实际效果

来自调试会话（2025-10-03）：
- 修复了 3 个文件中的 15 个不稳定测试
- 通过率：60% → 100%
- 执行时间：快了 40%
- 不再有竞态条件
