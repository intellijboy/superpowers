---
name: requesting-code-review
description: 当完成任务、实现主要功能或合并前使用，以验证工作符合要求
---

# 请求代码审查

派遣 superpowers:code-reviewer 子代理在问题级联之前捕获它们。审查者获得精确编写的评估上下文 — 永远不是你会话的历史。这让审查者专注于工作产品，而不是你的思考过程，并为你自己的持续工作保留上下文。

**核心原则：** 尽早审查，经常审查。

## 何时请求审查

**强制性：**
- 子代理驱动开发中的每个任务后
- 完成主要功能后
- 合并到 main 之前

**可选但有价值：**
- 当卡住时（新鲜视角）
- 重构之前（基线检查）
- 修复复杂 bug 后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣 code-reviewer 子代理：**

使用 Task 工具配合 superpowers:code-reviewer 类型，填写 `code-reviewer.md` 中的模板

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 你刚刚构建的内容
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交
- `{DESCRIPTION}` - 简要摘要

**3. 根据反馈行动：**
- 立即修复严重问题
- 在继续之前修复重要问题
- 记录次要问题以供以后
- 如果审查者错了则驳回（附理由）

## 示例

```
[刚完成任务 2：添加验证函数]

你：让我在继续之前请求代码审查。

BASE_SHA=$(git log --oneline | grep "任务 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派遣 superpowers:code-reviewer 子代理]
  WHAT_WAS_IMPLEMENTED: 对话索引的验证和修复函数
  PLAN_OR_REQUIREMENTS: 来自 docs/superpowers/plans/deployment-plan.md 的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，包含 4 种问题类型

[子代理返回]：
  优点：架构清晰，真实测试
  问题：
    重要：缺少进度指示器
    次要：报告间隔的魔术数字 (100)
  评估：准备继续

你：[修复进度指示器]
[继续任务 3]
```

## 与工作流集成

**子代理驱动开发：**
- 每个任务后审查
- 在问题累积之前捕获它们
- 在移动到下一个任务之前修复

**执行计划：**
- 每批次（3 个任务）后审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 红旗

**永远不要：**
- 因为"这很简单"而跳过审查
- 忽略严重问题
- 在未修复重要问题的情况下继续
- 与有效的技术反馈争论

**如果审查者错了：**
- 用技术理由驳回
- 展示证明它有效的代码/测试
- 请求澄清

参见模板：requesting-code-review/code-reviewer.md
