# 测试反模式

**在以下情况下加载此参考：**编写或更改测试、添加 mock，或者想要向生产代码添加仅测试方法时。

## 概述

测试必须验证真实行为，而不是 mock 行为。Mock 是隔离的手段，而不是被测试的东西。

**核心原则：**测试代码做什么，而不是 mock 做什么。

**遵循严格的 TDD 可以防止这些反模式。**

## 铁律

```
1. 永远不要测试 mock 行为
2. 永远不要向生产类添加仅测试方法
3. 永远不要在理解依赖项之前进行 mock
```

## 反模式 1：测试 Mock 行为

**违规：**
```typescript
// ❌ 不推荐：测试 mock 存在
test('渲染侧边栏', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么这是错误的：**
- 你正在验证 mock 有效，而不是组件有效
- 当 mock 存在时测试通过，不存在时失败
- 对真实行为没有任何说明

**你的 人类搭档的纠正：**"我们是在测试 mock 的行为吗？"

**修复：**
```typescript
// ✅ 推荐：测试真实组件或不 mock 它
test('渲染侧边栏', () => {
  render(<Page />);  // 不要 mock 侧边栏
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// 或者如果侧边栏必须为了隔离而 mock：
// 不要断言 mock - 测试侧边栏存在时的页面行为
```

### 闸门函数

```
在对任何 mock 元素断言之前：
  询问："我是在测试真实组件行为还是仅仅 mock 存在？"

  如果测试 mock 存在：
    停下 - 删除断言或取消 mock 组件

  改为测试真实行为
```

## 反模式 2：生产中的仅测试方法

**违规：**
```typescript
// ❌ 不推荐：destroy() 仅在测试中使用
class Session {
  async destroy() {  // 看起来像生产 API！
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... 清理
  }
}

// 在测试中
afterEach(() => session.destroy());
```

**为什么这是错误的：**
- 生产类被仅测试代码污染
- 如果在生产中意外调用很危险
- 违反 YAGNI 和关注点分离
- 混淆对象生命周期与实体生命周期

**修复：**
```typescript
// ✅ 推荐：测试工具处理测试清理
// Session 没有 destroy() - 在生产中它是无状态的

// 在 test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// 在测试中
afterEach(() => cleanupSession(session));
```

### 闸门函数

```
在向生产类添加任何方法之前：
  询问："这是否仅由测试使用？"

  如果是：
    停下 - 不要添加它
    将其放入测试工具中

  询问："此类是否拥有此资源的生命周期？"

  如果否：
    停下 - 错误的类用于此方法
```

## 反模式 3：在不理解的情况下 Mock

**违规：**
```typescript
// ❌ 不推荐：Mock 破坏测试逻辑
test('检测重复服务器', () => {
  // Mock 阻止了测试依赖的配置写入！
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // 应该抛出 - 但不会！
});
```

**为什么这是错误的：**
- Mock 方法具有测试依赖的副作用（写入配置）
- 过度 mock 以"安全"会破坏实际行为
- 测试因错误原因通过或神秘地失败

**修复：**
```typescript
// ✅ 推荐：在正确级别 mock
test('检测重复服务器', () => {
  // Mock 慢的部分，保留测试需要的行为
  vi.mock('MCPServerManager'); // 仅 mock 慢的服务器启动

  await addServer(config);  // 配置已写入
  await addServer(config);  // 检测到重复 ✓
});
```

### 闸门函数

```
在 mock 任何方法之前：
  停下 - 还不要 mock

  1. 询问："真实方法有什么副作用？"
  2. 询问："此测试是否依赖其中任何副作用？"
  3. 询问："我完全理解此测试需要什么吗？"

  如果依赖副作用：
    在较低级别 mock（实际的慢/外部操作）
    或使用保留必要行为的测试替身
    而不是测试依赖的高级方法

  如果不确定测试依赖什么：
    先使用真实实现运行测试
    观察实际需要发生什么
    然后在正确的级别添加最少的 mock

  危险信号：
    - "我会 mock 这个以保安全"
    - "这可能很慢，最好 mock 它"
    - 在不理解依赖链的情况下进行 mock
```

## 反模式 4：不完整的 Mock

**违规：**
```typescript
// ❌ 不推荐：部分 mock - 只有你认为需要的字段
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // 缺失：下游代码使用的元数据
};

// 后来：当代码访问 response.metadata.requestId 时中断
```

**为什么这是错误的：**
- **部分 mock 隐藏结构假设** - 你只 mock 了你知道的字段
- **下游代码可能依赖于你没有包含的字段** - 静默失败
- **测试通过但集成失败** - mock 不完整，真实 API 完整
- **虚假信心** - 测试对真实行为没有任何证明

**铁律：**模拟现实中的完整数据结构，而不仅仅是你的直接测试使用的字段。

**修复：**
```typescript
// ✅ 推荐：反映真实 API 的完整性
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // 真实 API 返回的所有字段
};
```

### 闸门函数

```
在创建 mock 响应之前：
  检查："真实 API 响应包含哪些字段？"

  操作：
    1. 检查文档/示例中的实际 API 响应
    2. 包含系统可能在下游使用的所有字段
    3. 验证 mock 完全匹配真实响应模式

  关键：
    如果你正在创建 mock，你必须理解整个结构
    当代码依赖于省略的字段时，部分 mock 会静默失败

  如果不确定：包含所有记录的字段
```

## 反模式 5：集成测试作为事后补充

**违规：**
```
✅ 实现完成
❌ 没有编写测试
"准备测试"
```

**为什么这是错误的：**
- 测试是实现的一部分，而不是可选的后续
- TDD 会捕获这一点
- 没有测试就不能声称完成

**修复：**
```
TDD 周期：
1. 编写失败的测试
2. 实现以通过
3. 重构
4. 然后声称完成
```

## 当 Mock 变得太复杂

**警告信号：**
- Mock 设置比测试逻辑长
- Mock 所有东西以使测试通过
- Mock 缺少真实组件具有的方法
- 当 mock 更改时测试中断

**你的 人类搭档的问题：**"我们需要在这里使用 mock 吗？"

**考虑：**具有真实组件的集成测试通常比复杂的 mock 更简单

## TDD 防止这些反模式

**为什么 TDD 有帮助：**
1. **先编写测试** → 迫使你思考实际测试的内容
2. **看着它失败** → 确认测试测试真实行为，而不是 mock
3. **最少的实现** → 没有仅测试方法混入
4. **真实依赖** → 在 mock 之前你看到测试实际需要什么

**如果你正在测试 mock 行为，你就违反了 TDD** - 你在没有先看着测试针对真实代码失败的情况下添加了 mock。

## 快速参考

| 反模式 | 修复 |
|--------------|-----|
| 断言 mock 元素 | 测试真实组件或取消 mock |
| 生产中的仅测试方法 | 移至测试工具 |
| 在不理解的情况下 mock | 首先理解依赖项，最少 mock |
| 不完整的 mock | 完全反映真实 API |
| 测试作为事后补充 | TDD - 测试优先 |
| 过于复杂的 mock | 考虑集成测试 |

## 危险信号

- 断言检查 `*-mock` 测试 ID
- 仅在测试文件中调用的方法
- Mock 设置占测试的 50% 以上
- 删除 mock 时测试失败
- 无法解释为什么需要 mock
- "为了安全而"mock

## 底线

**Mock 是隔离的工具，而不是测试的东西。**

如果 TDD 揭示你正在测试 mock 行为，你就错了。

修复：测试真实行为或质疑为什么要 mock。
