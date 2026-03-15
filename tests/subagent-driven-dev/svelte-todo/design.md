# Svelte Todo List - 设计

## 概述

使用 Svelte 构建的简单 todo 列表应用。支持创建、完成和删除 todo，具有 localStorage 持久化。

## 功能

- 添加新 todo
- 将 todo 标记为完成/未完成
- 删除 todo
- 按以下方式过滤：All / Active / Completed
- 清除所有已完成的 todo
- 持久化到 localStorage
- 显示剩余项目计数

## 用户界面

```
┌─────────────────────────────────────────┐
│  Svelte Todos                           │
├─────────────────────────────────────────┤
│  [________________________] [Add]       │
├─────────────────────────────────────────┤
│  [ ] Buy groceries                  [x] │
│  [✓] Walk the dog                   [x] │
│  [ ] Write code                     [x] │
├─────────────────────────────────────────┤
│  2 items left                           │
│  [All] [Active] [Completed]  [Clear ✓]  │
└─────────────────────────────────────────┘
```

## 组件

```
src/
  App.svelte           # 主应用，状态管理
  lib/
    TodoInput.svelte   # 文本输入 + 添加按钮
    TodoList.svelte    # 列表容器
    TodoItem.svelte    # 单个 todo，带复选框、文本、删除
    FilterBar.svelte   # 过滤按钮 + 清除已完成
    store.ts           # todo 的 Svelte store
    storage.ts         # localStorage 持久化
```

## 数据模型

```typescript
interface Todo {
  id: string;        // UUID
  text: string;      // Todo 文本
  completed: boolean;
}

type Filter = 'all' | 'active' | 'completed';
```

## 验收标准

1. 可以通过输入并按 Enter 或点击 Add 来添加 todo
2. 可以通过点击复选框切换 todo 完成状态
3. 可以通过点击 X 按钮删除 todo
4. 过滤按钮显示正确的 todo 子集
5. "X items left" 显示未完成 todo 的计数
6. "Clear completed" 删除所有已完成的 todo
7. Todo 在页面刷新后保留（localStorage）
8. 空状态显示有用的消息
9. 所有测试通过
