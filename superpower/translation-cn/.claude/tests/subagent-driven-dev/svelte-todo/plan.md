# Svelte 待办事项列表 - 实现计划

使用 `superpowers:subagent-driven-development` skill 执行此计划。

## 上下文

使用 Svelte 构建一个待办事项列表应用。完整规范见 `design.md`。

## 任务

### 任务 1：项目搭建

使用 Vite 创建 Svelte 项目。

**执行：**
- 运行 `npm create vite@latest . -- --template svelte-ts`
- 使用 `npm install` 安装依赖
- 验证开发服务器可正常运行
- 清理 App.svelte 中的默认 Vite 模板内容

**验证：**
- `npm run dev` 启动服务器
- 应用显示最小的 "Svelte Todos" 标题
- `npm run build` 成功

---

### 任务 2：Todo Store

创建用于待办事项状态管理的 Svelte store。

**执行：**
- 创建 `src/lib/store.ts`
- 定义 `Todo` 接口，包含 id、text、completed
- 创建初始化为空数组的可写 store
- 导出函数：`addTodo(text)`、`toggleTodo(id)`、`deleteTodo(id)`、`clearCompleted()`
- 创建 `src/lib/store.test.ts`，为每个函数编写测试

**验证：**
- 测试通过：`npm run test`（如需要，安装 vitest）

---

### 任务 3：localStorage 持久化

为待办事项添加持久化层。

**执行：**
- 创建 `src/lib/storage.ts`
- 实现 `loadTodos(): Todo[]` 和 `saveTodos(todos: Todo[])`
- 优雅处理 JSON 解析错误（返回空数组）
- 与 store 集成：初始化时加载，变更时保存
- 为加载/保存/错误处理添加测试

**验证：**
- 测试通过
- 手动测试：添加待办事项，刷新页面，待办事项仍然存在

---

### 任务 4：TodoInput 组件

创建用于添加待办事项的输入组件。

**执行：**
- 创建 `src/lib/TodoInput.svelte`
- 文本输入绑定到本地状态
- Add 按钮调用 `addTodo()` 并清空输入
- Enter 键也可提交
- 输入为空时禁用 Add 按钮
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染输入框和按钮

---

### 任务 5：TodoItem 组件

创建单个待办事项组件。

**执行：**
- 创建 `src/lib/TodoItem.svelte`
- Props：`todo: Todo`
- 复选框切换完成状态（调用 `toggleTodo`）
- 已完成时文本显示删除线
- 删除按钮 (X) 调用 `deleteTodo`
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染复选框、文本和删除按钮

---

### 任务 6：TodoList 组件

创建列表容器组件。

**执行：**
- 创建 `src/lib/TodoList.svelte`
- Props：`todos: Todo[]`
- 为每个待办事项渲染 TodoItem
- 列表为空时显示 "No todos yet"
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染 TodoItem 列表

---

### 任务 7：FilterBar 组件

创建筛选和状态栏组件。

**执行：**
- 创建 `src/lib/FilterBar.svelte`
- Props：`todos: Todo[]`、`filter: Filter`、`onFilterChange: (f: Filter) => void`
- 显示计数："X items left"（未完成数量）
- 三个筛选按钮：All、Active、Completed
- 当前激活的筛选按钮视觉上高亮
- "Clear completed" 按钮（无可完成事项时隐藏）
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染计数、筛选按钮和清除按钮

---

### 任务 8：App 集成

在 App.svelte 中连接所有组件。

**执行：**
- 导入所有组件和 store
- 添加筛选状态（默认：'all'）
- 根据筛选状态计算筛选后的待办事项
- 渲染：标题、TodoInput、TodoList、FilterBar
- 向各组件传递合适的 props

**验证：**
- 应用渲染所有组件
- 添加待办事项功能正常
- 切换功能正常
- 删除功能正常

---

### 任务 9：筛选功能

确保筛选端到端正常工作。

**执行：**
- 验证筛选按钮切换显示的待办事项
- 'all' 显示所有待办事项
- 'active' 仅显示未完成事项
- 'completed' 仅显示已完成事项
- Clear completed 移除已完成事项并在需要时重置筛选
- 添加集成测试

**验证：**
- 筛选测试通过
- 手动验证所有筛选状态

---

### 任务 10：样式与打磨

添加 CSS 样式以提升可用性。

**执行：**
- 为应用添加样式以匹配设计模型
- 已完成事项显示删除线和柔和颜色
- 当前激活的筛选按钮高亮
- 输入框具有焦点样式
- 删除按钮在悬停时显示（移动端始终显示）
- 响应式布局

**验证：**
- 应用视觉上可用
- 样式不影响功能

---

### 任务 11：端到端测试

为完整的用户操作流程添加 Playwright 测试。

**执行：**
- 安装 Playwright：`npm init playwright@latest`
- 创建 `tests/todo.spec.ts`
- 测试流程：
  - 添加待办事项
  - 完成待办事项
  - 删除待办事项
  - 筛选待办事项
  - 清除已完成事项
  - 持久化（添加、刷新、验证）

**验证：**
- `npx playwright test` 通过

---

### 任务 12：README

编写项目文档。

**执行：**
- 创建 `README.md`，包含：
  - 项目描述
  - 安装步骤：`npm install`
  - 开发：`npm run dev`
  - 测试：`npm test` 和 `npx playwright test`
  - 构建：`npm run build`

**验证：**
- README 准确描述项目
- 指示步骤可正常工作
