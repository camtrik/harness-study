# Svelte Todo List - 实施计划

使用 `superpowers:subagent-driven-development` skill 执行此计划。

## 上下文

使用 Svelte 构建待办事项列表应用。完整规范参见 `design.md`。

## 任务

### 任务 1: 项目设置

使用 Vite 创建 Svelte 项目。

**操作:**
- 运行 `npm create vite@latest . -- --template svelte-ts`
- 使用 `npm install` 安装依赖
- 验证 dev server 工作
- 从 App.svelte 中清理默认的 Vite 模板内容

**验证:**
- `npm run dev` 启动 server
- App 显示最小的 "Svelte Todos" 标题
- `npm run build` 成功

---

### 任务 2: Todo Store

创建用于待办事项状态管理的 Svelte store。

**操作:**
- 创建 `src/lib/store.ts`
- 定义 `Todo` 接口,包含 id、text、completed
- 创建带有初始空数组的可写 store
- 导出函数: `addTodo(text)`, `toggleTodo(id)`, `deleteTodo(id)`, `clearCompleted()`
- 创建 `src/lib/store.test.ts`,为每个函数添加测试

**验证:**
- 测试通过: `npm run test` (如需要则安装 vitest)

---

### 任务 3: localStorage 持久化

为待办事项添加持久化层。

**操作:**
- 创建 `src/lib/storage.ts`
- 实现 `loadTodos(): Todo[]` 和 `saveTodos(todos: Todo[])`
- 优雅地处理 JSON 解析错误(返回空数组)
- 与 store 集成: 初始化时加载,更改时保存
- 为加载/保存/错误处理添加测试

**验证:**
- 测试通过
- 手动测试: 添加待办事项,刷新页面,待办事项持久化

---

### 任务 4: TodoInput 组件

创建用于添加待办事项的输入组件。

**操作:**
- 创建 `src/lib/TodoInput.svelte`
- 文本输入绑定到本地状态
- 添加按钮调用 `addTodo()` 并清除输入
- Enter 键也提交
- 输入为空时禁用 Add 按钮
- 添加组件测试

**验证:**
- 测试通过
- 组件渲染输入和按钮

---

### 任务 5: TodoItem 组件

创建单个待办事项组件。

**操作:**
- 创建 `src/lib/TodoItem.svelte`
- Props: `todo: Todo`
- 复选框切换完成状态(调用 `toggleTodo`)
- 完成时文本带有删除线
- 删除按钮(X)调用 `deleteTodo`
- 添加组件测试

**验证:**
- 测试通过
- 组件渲染复选框、文本、删除按钮

---

### 任务 6: TodoList 组件

创建列表容器组件。

**操作:**
- 创建 `src/lib/TodoList.svelte`
- Props: `todos: Todo[]`
- 为每个待办事项渲染 TodoItem
- 空时显示 "No todos yet"
- 添加组件测试

**验证:**
- 测试通过
- 组件渲染 TodoItem 列表

---

### 任务 7: FilterBar 组件

创建过滤和状态栏组件。

**操作:**
- 创建 `src/lib/FilterBar.svelte`
- Props: `todos: Todo[]`, `filter: Filter`, `onFilterChange: (f: Filter) => void`
- 显示计数: "X items left" (未完成计数)
- 三个过滤按钮: All、Active、Completed
- 活跃过滤器视觉高亮
- "Clear completed" 按钮(无已完成待办事项时隐藏)
- 添加组件测试

**验证:**
- 测试通过
- 组件渲染计数、过滤器、清除按钮

---

### 任务 8: App 集成

在 App.svelte 中连接所有组件。

**操作:**
- 导入所有组件和 store
- 添加过滤状态(默认: 'all')
- 基于过滤状态计算过滤的待办事项
- 渲染: 标题、TodoInput、TodoList、FilterBar
- 为每个组件传递适当的 props

**验证:**
- App 渲染所有组件
- 添加待办事项工作
- 切换工作
- 删除工作

---

### 任务 9: 过滤功能

确保端到端过滤工作。

**操作:**
- 验证过滤按钮改变显示的待办事项
- 'all' 显示所有待办事项
- 'active' 仅显示未完成的待办事项
- 'completed' 仅显示已完成的待办事项
- Clear completed 删除已完成的待办事项,如需要则重置过滤器
- 添加集成测试

**验证:**
- 过滤测试通过
- 手动验证所有过滤状态

---

### 任务 10: 样式和打磨

添加 CSS 样式以提高可用性。

**操作:**
- 为 app 样式以匹配设计 mockup
- 已完成的待办事项有删除线和柔和的颜色
- 活跃过滤按钮高亮
- 输入有焦点样式
- 删除按钮在悬停时出现(或在移动端始终显示)
- 响应式布局

**验证:**
- App 视觉上可用
- 样式不破坏功能

---

### 任务 11: 端到端测试

为完整的用户流程添加 Playwright 测试。

**操作:**
- 安装 Playwright: `npm init playwright@latest`
- 创建 `tests/todo.spec.ts`
- 测试流程:
  - 添加待办事项
  - 完成待办事项
  - 删除待办事项
  - 过滤待办事项
  - 清除已完成
  - 持久化(添加、重新加载、验证)

**验证:**
- `npx playwright test` 通过

---

### 任务 12: README

记录项目。

**操作:**
- 创建 `README.md`,包含:
  - 项目描述
  - 设置: `npm install`
  - 开发: `npm run dev`
  - 测试: `npm test` 和 `npx playwright test`
  - 构建: `npm run build`

**验证:**
- README 准确描述项目
- 指令工作
