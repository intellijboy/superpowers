# 文档评审系统设计

## 概述

向 superpowers 工作流添加两个新的评审阶段：

1. **Spec 文档评审** - brainstorming 之后，writing-plans 之前
2. **计划文档评审** - writing-plans 之后，实现之前

两者都遵循实现评审所使用的迭代循环模式。

## Spec 文档评审器

**目的：** 验证 spec 是否完整、一致，并准备好进行实现规划。

**位置：** `skills/brainstorming/spec-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 检查要点 |
|----------|------------------|
| 完整性 | TODOs、占位符、"TBD"、不完整的章节 |
| 覆盖范围 | 缺失的错误处理、边界情况、集成点 |
| 一致性 | 内部矛盾、冲突的需求 |
| 清晰度 | 模糊的需求 |
| YAGNI | 未请求的功能、过度工程 |

**输出格式：**
```
## Spec 评审

**状态：** 通过 | 发现问题

**问题（如有）：**
- [章节 X]: [问题] - [为什么重要]

**建议（参考）：**
- [不阻塞通过的建议]
```

**评审循环：** 发现问题 -> brainstorming agent 修复 -> 重新评审 -> 重复直到通过。

**调度机制：** 使用 Task 工具，设置 `subagent_type: general-purpose`。评审器提示模板提供完整的提示。Brainstorming 技能的控制器调度评审器。

## 计划文档评审器

**目的：** 验证计划是否完整、与 spec 匹配，并具有正确的任务分解。

**位置：** `skills/writing-plans/plan-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 检查要点 |
|----------|------------------|
| 完整性 | TODOs、占位符、不完整的任务 |
| Spec 对齐 | 计划覆盖 spec 需求，无范围蔓延 |
| 任务分解 | 任务原子化，边界清晰 |
| 任务语法 | 任务和步骤使用复选框语法 |
| Chunk 大小 | 每个 chunk 少于 1000 行 |

**Chunk 定义：** Chunk 是计划文档中任务的逻辑分组，由 `## Chunk N: <name>` 标题分隔。Writing-plans 技能根据逻辑阶段（如"基础"、"核心功能"、"集成"）创建这些边界。每个 chunk 应足够独立，可以单独评审。

**Spec 对齐验证：** 评审器同时接收：
1. 计划文档（或当前 chunk）
2. Spec 文档的路径供参考

评审器读取两者并比较需求覆盖情况。

**输出格式：** 与 spec 评审器相同，但范围限定在当前 chunk。

**评审过程（逐 chunk）：**
1. Writing-plans 创建 chunk N
2. 控制器调度 plan-document-reviewer，传入 chunk N 内容和 spec 路径
3. 评审器读取 chunk 和 spec，返回裁决
4. 如果有问题：writing-plans agent 修复 chunk N，转到步骤 2
5. 如果通过：继续 chunk N+1
6. 重复直到所有 chunk 通过

**调度机制：** 与 spec 评审器相同 - 使用 Task 工具，设置 `subagent_type: general-purpose`。

## 更新后的工作流

```
brainstorming -> spec -> SPEC 评审循环 -> writing-plans -> plan -> PLAN 评审循环 -> implementation
```

**Spec 评审循环：**
1. Spec 完成
2. 调度评审器
3. 如果有问题：修复 -> 转到 2
4. 如果通过：继续

**计划评审循环：**
1. Chunk N 完成
2. 为 chunk N 调度评审器
3. 如果有问题：修复 -> 转到 2
4. 如果通过：下一个 chunk 或实现

## Markdown 任务语法

任务和步骤使用复选框语法：

```markdown
- [ ] ### Task 1: 名称

- [ ] **Step 1:** 描述
  - 文件: 路径
  - 命令: cmd
```

## 错误处理

**评审循环终止：**
- 无硬性迭代限制 - 循环继续直到评审器通过
- 如果循环超过 5 次迭代，控制器应向人类寻求指导
- 人类可以选择：继续迭代、带已知问题通过、或中止

**分歧处理：**
- 评审器是建议性的 - 它们标记问题但不阻塞
- 如果 agent 认为评审器反馈不正确，应在修复中解释原因
- 如果同一问题在 3 次迭代后仍有分歧，向人类汇报

**格式错误的评审器输出：**
- 控制器应验证评审器输出具有必需字段（状态、问题（如适用））
- 如果格式错误，重新调度评审器并说明预期格式
- 2 次格式错误响应后，向人类汇报

## 需要更改的文件

**新文件：**
- `skills/brainstorming/spec-document-reviewer-prompt.md`
- `skills/writing-plans/plan-document-reviewer-prompt.md`

**修改的文件：**
- `skills/brainstorming/SKILL.md` - 在 spec 写入后添加评审循环
- `skills/writing-plans/SKILL.md` - 添加逐 chunk 评审循环，更新任务语法示例
