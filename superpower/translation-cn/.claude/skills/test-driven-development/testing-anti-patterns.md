# 测试反模式

**何时加载此参考：** 编写或修改测试时、添加 mock 时，或想要向生产代码添加仅供测试的方法时。

## 概述

测试必须验证真实行为，而非 mock 行为。mock 是隔离的手段，而不是被测试的对象。

**核心原则：** 测试代码做什么，而不是测试 mock 做什么。

**遵循严格的 TDD 可以防止这些反模式。**

## 铁律

```
1. 永远不要测试 mock 行为
2. 永远不要向生产类添加仅供测试的方法
3. 永远不要在不理解依赖的情况下 mock
```

## 反模式 1：测试 Mock 行为

**违规示例：**
```typescript
// ❌ 坏：测试 mock 是否存在
test('渲染侧边栏', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么这是错的：**
- 你验证的是 mock 是否有效，而不是组件是否有效
- mock 存在时测试通过，不存在时测试失败
- 对真实行为什么都没告诉你

**你的人类搭档的校正：** "我们是在测试 mock 的行为吗？"

**修复方案：**
```typescript
// ✅ 好：测试真实组件，或者不要 mock 它
test('渲染侧边栏', () => {
  render(<Page />);  // 不要 mock 侧边栏
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// 或者如果出于隔离必须 mock 侧边栏：
// 不要对 mock 做断言——测试 Page 在侧边栏存在时的行为
```

### 门控函数

```
在对任何 mock 元素做断言之前：
  问："我是在测试真实组件的行为还是仅仅测试 mock 的存在？"

  如果在测试 mock 的存在：
    停下来——删除该断言或者取消对该组件的 mock

  改为测试真实行为
```

## 反模式 2：生产代码中的仅供测试方法

**违规示例：**
```typescript
// ❌ 坏：destroy() 只在测试中使用
class Session {
  async destroy() {  // 看起来像生产 API！
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... 清理
  }
}

// 在测试中
afterEach(() => session.destroy());
```

**为什么这是错的：**
- 生产类被仅供测试的代码污染
- 如果在生产环境中意外调用会危险
- 违反 YAGNI 和关注点分离
- 混淆了对象生命周期和实体生命周期

**修复方案：**
```typescript
// ✅ 好：测试工具处理测试清理
// Session 没有 destroy()——它在生产环境中是无状态的

// 在 test-utils/ 中
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// 在测试中
afterEach(() => cleanupSession(session));
```

### 门控函数

```
在向生产类添加任何方法之前：
  问："这个方法是仅供测试使用的吗？"

  如果是：
    停下来——不要添加它
    放在测试工具中

  问："这个类是否拥有此资源的生命周期？"

  如果不是：
    停下来——这个方法不应该放在这个类里
```

## 反模式 3：不理解就 Mock

**违规示例：**
```typescript
// ❌ 坏：mock 破坏了测试逻辑
test('检测重复服务器', () => {
  // mock 阻止了测试依赖的配置写入！
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // 应该抛出——但不会了！
});
```

**为什么这是错的：**
- 被 mock 的方法有测试依赖的副作用（写入配置）
- 为了"安全"过度 mock 破坏了实际行为
- 测试因错误原因通过，或者神秘地失败

**修复方案：**
```typescript
// ✅ 好：在正确的层级 mock
test('检测重复服务器', () => {
  // mock 慢的部分，保留测试需要的行为
  vi.mock('MCPServerManager'); // 只 mock 慢的服务器启动

  await addServer(config);  // 配置已写入
  await addServer(config);  // 重复检测 ✓
});
```

### 门控函数

```
在 mock 任何方法之前：
  停下来——先不要 mock

  1. 问："真实方法有哪些副作用？"
  2. 问："这个测试是否依赖其中任何一个副作用？"
  3. 问："我是否完全理解这个测试需要什么？"

  如果依赖副作用：
    在更低层级 mock（实际慢/外部的操作）
    或者使用 test double 来保留必要的行为
    而不是 mock 测试依赖的高层方法

  如果不确定测试依赖什么：
    先用真实实现运行测试
    观察实际需要发生什么
    然后在正确的层级添加最小的 mock

  危险信号：
    - "我先 mock 这个保稳"
    - "这可能慢，最好 mock 掉"
    - 不理解依赖链就 mock
```

## 反模式 4：不完整的 Mock

**违规示例：**
```typescript
// ❌ 坏：部分 mock——只包含你认为需要的字段
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // 缺失：下游代码使用的 metadata
};

// 之后：当代码访问 response.metadata.requestId 时崩溃
```

**为什么这是错的：**
- **部分 mock 隐藏了结构假设**——你只 mock 了你已知的字段
- **下游代码可能依赖你没有包含的字段**——静默失败
- **测试通过但集成失败**——mock 不完整，真实 API 完整
- **虚假信心**——测试对真实行为什么都证明不了

**铁律：** mock 完整的数据结构，就如它在现实中存在的那样，而不仅仅是你当前测试用到的字段。

**修复方案：**
```typescript
// ✅ 好：镜像真实 API 的完整性
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // 真实 API 返回的所有字段
};
```

### 门控函数

```
在创建 mock 响应之前：
  检查："真实 API 响应包含哪些字段？"

  操作：
    1. 从文档/示例中检查实际 API 响应
    2. 包含系统下游可能使用的所有字段
    3. 验证 mock 完全匹配真实响应模式

  关键：
    如果你要创建 mock，你必须理解整个结构
    当代码依赖被省略的字段时，部分 mock 会静默失败

  如果不确定：包含所有已文档化的字段
```

## 反模式 5：事后集成测试

**违规示例：**
```
✅ 实现完成
❌ 没有写测试
"准备测试"
```

**为什么这是错的：**
- 测试是实现的一部分，不是可选的后续步骤
- TDD 本可以避免这种情况
- 没有测试不能声称完成

**修复方案：**
```
TDD 循环：
1. 写失败的测试
2. 实现使其通过
3. 重构
4. 然后才声称完成
```

## 当 Mock 变得过于复杂时

**警告信号：**
- mock 准备代码比测试逻辑还长
- 为了让测试通过 mock 了所有东西
- mock 缺少真实组件拥有的方法
- mock 变更时测试就崩溃

**你的人类搭档的问题：** "我们这里需要用 mock 吗？"

**考虑：** 使用真实组件的集成测试通常比复杂的 mock 更简单

## TDD 防止这些反模式

**为什么 TDD 有帮助：**
1. **先写测试** → 迫使你思考你实际在测试什么
2. **看它失败** → 确认测试的是真实行为，而不是 mock
3. **最小化实现** → 不会有仅供测试的方法渗入
4. **真实依赖** → 你在 mock 之前就能看到测试实际需要什么

**如果你在测试 mock 行为，你就违反了 TDD**——你在未看到测试对真实代码失败之前就添加了 mock。

## 快速参考

| 反模式 | 修复方案 |
|--------------|-----|
| 对 mock 元素做断言 | 测试真实组件或取消 mock |
| 生产代码中的仅供测试方法 | 移到测试工具中 |
| 不理解就 mock | 先理解依赖，最小化 mock |
| 不完整的 mock | 完全镜像真实 API |
| 事后测试 | TDD——测试先行 |
| 过度复杂的 mock | 考虑集成测试 |

## 危险信号

- 断言检查 `*-mock` 测试 ID
- 方法只在测试文件中调用
- mock 准备代码占测试的 50% 以上
- 移除 mock 后测试失败
- 无法解释为什么需要 mock
- "只是为了安全"而 mock

## 底线

**mock 是隔离的工具，不是被测试的对象。**

如果 TDD 揭示你在测试 mock 行为，那你就走错了。

修复：测试真实行为，或者质疑你为什么要 mock。
