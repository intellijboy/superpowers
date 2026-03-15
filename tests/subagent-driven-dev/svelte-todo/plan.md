# Svelte Todo List - 实现计划

使用 `superpowers:subagent-driven-development` skill 执行此计划。

## 上下文

使用 Svelte 构建一个 todo 列表应用。完整规格参见 `design.md`。

## 任务

### Task 1: 项目设置

使用 Vite 创建 Svelte 项目。

**要做：**
- 运行 `npm create vite@latest . -- --template svelte-ts`
- 使用 `npm install` 安装依赖
- 验证开发服务器可以工作
- 清理 App.svelte 中的默认 Vite 模板内容

**验证：**
- `npm run dev` 启动服务器
- 应用显示最小的"Svelte Todos"标题
- `npm run build` 成功

---

### Task 2: Todo Store

创建用于 todo 状态管理的 Svelte store。

**要做：**
- 创建 `src/lib/store.ts`
- 定义包含 id、text、completed 的 `Todo` 接口
- 创建带有初始空数组的 writable store
- 导出函数：`addTodo(text)`、`toggleTodo(id)`、`deleteTodo(id)`、`clearCompleted()`
- 创建 `src/lib/store.test.ts`，包含每个函数的测试

**验证：**
- 测试通过：`npm run test`（如需要安装 vitest）

---

### Task 3: TodoInput 组件

创建用于添加 todo 的输入组件。

**要做：**
- 创建 `src/lib/TodoInput.svelte`
- 文本输入绑定到本地状态
- 添加按钮调用 `addTodo()` 并清空输入
- Enter 键也提交
- 输入为空时禁用添加按钮
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染输入和按钮

---

### Task 4: TodoItem 组件

创建单个 todo 项组件。

**要做：**
- 创建 `src/lib/TodoItem.svelte`
- Props：`todo: Todo`
- 复选框切换完成状态（调用 `toggleTodo`）
- 完成时文本有删除线
- 删除按钮（X）调用 `deleteTodo`
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染复选框、文本、删除按钮

---

### Task 5: TodoList 组件

创建列表容器组件。

**要做：**
- 创建 `src/lib/TodoList.svelte`
- Props：`todos: Todo[]`
- 为每个 todo 渲染 TodoItem
- 空时显示"No todos yet"
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染 TodoItem 列表

---

### Task 6: FilterBar 组件

创建过滤器和状态栏组件。

**要做：**
- 创建 `src/lib/FilterBar.svelte`
- Props：`todos: Todo[]`、`filter: Filter`、`onFilterChange: (f: Filter) => void`
- 显示计数："X items left"（未完成计数）
- 三个过滤按钮：All、Active、Completed
- 当前过滤器视觉高亮
- "Clear completed"按钮（没有已完成 todo 时隐藏）
- 添加组件测试

**验证：**
- 测试通过
- 组件渲染计数、过滤器、清除按钮

---

### Task 7: 应用集成

在 App.svelte 中将所有组件连接起来。

**要做：**
- 导入所有组件和 store
- 添加过滤器状态（默认：'all'）
- 根据过滤器状态计算过滤后的 todo
- 渲染：标题、TodoInput、TodoList、FilterBar
- 向每个组件传递适当的 props

**验证：**
- 应用渲染所有组件
- 添加 todo 可工作
- 切换可工作
- 删除可工作

---

### Task 8: 过滤功能

确保过滤端到端工作。

**要做：**
- 验证过滤器按钮改变显示的 todo
- 'all' 显示所有 todo
- 'active' 只显示未完成的 todo
- 'completed' 只显示已完成的 todo
- 清除已完成删除已完成的 todo，如需要重置过滤器
- 添加集成测试

**验证：**
- 过滤器测试通过
- 手动验证所有过滤器状态

---

### Task 9: 样式和打磨

添加 CSS 样式以提高可用性。

**要做：**
- 为应用设计样式以匹配设计原型
- 已完成的 todo 有删除线和柔和的颜色
- 当前过滤器按钮高亮
- 输入有焦点样式
- 删除按钮悬停时显示（或移动端始终显示）
- 响应式布局

**验证：**
- 应用视觉上可用
- 样式不破坏功能

---

### Task 10: 端到端测试

为完整用户流程添加 Playwright 测试。

**要做：**
- 安装 Playwright：`npm init playwright@latest`
- 创建 `tests/todo.spec.ts`
- 测试流程：
  - 添加 todo
  - 完成 todo
  - 删除 todo
  - 过滤 todo
  - 清除已完成
  - 持久化（添加、重载、验证）

**验证：**
- `npx playwright test` 通过

---

### Task 11: localStorage 持久化

为 todo 添加持久化。

**要做：**
- 创建 `src/lib/storage.ts`
- 实现 `loadTodos(): Todo[]` 和 `saveTodos(todos: Todo[])`
- 优雅处理 JSON 解析错误（返回空数组）
- 与 store 集成：初始化时加载，变化时保存
- 添加 load/save/错误处理的测试

**验证：**
- 测试通过
- 手动测试：添加 todo，刷新页面，todo 保留

---

### Task 12: README

记录项目。

**要做：**
- 创建 `README.md` 包含：
  - 项目描述
  - 设置：`npm install`
  - 开发：`npm run dev`
  - 测试：`npm test` 和 `npx playwright test`
  - 构建：`npm run build`

**验证：**
- README 准确描述项目
- 指令可工作
