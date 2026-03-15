# 文档评审系统实现计划

> **对于 agentic workers：** 必需：使用 superpowers:subagent-driven-development（如果 subagent 可用）或 superpowers:executing-plans 来实现此计划。

**目标：** 将 spec 和计划文档评审循环添加到 brainstorming 和 writing-plans 技能中。

**架构：** 在每个技能目录中创建评审器提示模板。修改技能文件，在文档创建后添加评审循环。使用 Task 工具配合 general-purpose subagent 来调度评审器。

**技术栈：** Markdown 技能文件，通过 Task 工具进行 subagent 调度

**规格文档：** docs/superpowers/specs/2026-01-22-document-review-system-design.md

---

## Chunk 1: Spec 文档评审器

此 chunk 将 spec 文档评审器添加到 brainstorming 技能。

### Task 1: 创建 Spec 文档评审器提示模板

**文件：**
- 创建：`skills/brainstorming/spec-document-reviewer-prompt.md`

- [ ] **Step 1:** 创建评审器提示模板文件

```markdown
# Spec 文档评审器提示模板

在调度 spec 文档评审器 subagent 时使用此模板。

**目的：** 验证 spec 是否完整、一致，并准备好进行实现规划。

**调度时机：** Spec 文档写入 docs/superpowers/specs/ 之后

```
Task tool (general-purpose):
  description: "Review spec document"
  prompt: |
    你是一个 spec 文档评审器。验证此 spec 是否完整并准备好进行规划。

    **待评审的 Spec：** [SPEC_FILE_PATH]

    ## 检查内容

    | 类别 | 检查要点 |
    |----------|------------------|
    | 完整性 | TODOs、占位符、"TBD"、不完整的章节 |
    | 覆盖范围 | 缺失的错误处理、边界情况、集成点 |
    | 一致性 | 内部矛盾、冲突的需求 |
    | 清晰度 | 模糊的需求 |
    | YAGNI | 未请求的功能、过度工程 |

    ## 关键检查

    特别注意：
    - 任何 TODO 标记或占位符文本
    - 说"稍后定义"或"等 X 完成后再 spec"的章节
    - 明显比其他章节不够详细的章节

    ## 输出格式

    ## Spec 评审

    **状态：** ✅ 通过 | ❌ 发现问题

    **问题（如有）：**
    - [章节 X]: [具体问题] - [为什么重要]

    **建议（参考）：**
    - [不阻塞通过的建议]
```

**评审器返回：** 状态、问题（如有）、建议
```

- [ ] **Step 2:** 验证文件创建正确

运行：`cat skills/brainstorming/spec-document-reviewer-prompt.md | head -20`
预期：显示标题和目的部分

- [ ] **Step 3:** 提交

```bash
git add skills/brainstorming/spec-document-reviewer-prompt.md
git commit -m "feat: add spec document reviewer prompt template"
```

---

### Task 2: 将评审循环添加到 Brainstorming 技能

**文件：**
- 修改：`skills/brainstorming/SKILL.md`

- [ ] **Step 1:** 读取当前的 brainstorming 技能

运行：`cat skills/brainstorming/SKILL.md`

- [ ] **Step 2:** 在"After the Design"之后添加评审循环部分

找到"After the Design"部分，在文档说明之后、实现设置之前添加一个新的"Spec 评审循环"部分：

```markdown
**Spec 评审循环：**
写入 spec 文档后：
1. 调度 spec-document-reviewer subagent（参见 spec-document-reviewer-prompt.md）
2. 如果 ❌ 发现问题：
   - 在 spec 文档中修复问题
   - 重新调度评审器
   - 重复直到 ✅ 通过
3. 如果 ✅ 通过：继续进行实现设置

**评审循环指导：**
- 编写 spec 的同一个 agent 修复它（保留上下文）
- 如果循环超过 5 次迭代，向人类寻求指导
- 评审器是建议性的 - 如果您认为反馈不正确，请解释不同意见
```

- [ ] **Step 3:** 验证更改

运行：`grep -A 15 "Spec 评审循环" skills/brainstorming/SKILL.md`
预期：显示新的评审循环部分

- [ ] **Step 4:** 提交

```bash
git add skills/brainstorming/SKILL.md
git commit -m "feat: add spec review loop to brainstorming skill"
```

---

## Chunk 2: 计划文档评审器

此 chunk 将计划文档评审器添加到 writing-plans 技能。

### Task 3: 创建计划文档评审器提示模板

**文件：**
- 创建：`skills/writing-plans/plan-document-reviewer-prompt.md`

- [ ] **Step 1:** 创建评审器提示模板文件

```markdown
# 计划文档评审器提示模板

在调度计划文档评审器 subagent 时使用此模板。

**目的：** 验证计划 chunk 是否完整、与 spec 匹配，并具有正确的任务分解。

**调度时机：** 每个计划 chunk 写入后

```
Task tool (general-purpose):
  description: "Review plan chunk N"
  prompt: |
    你是一个计划文档评审器。验证此计划 chunk 是否完整并准备好进行实现。

    **待评审的计划 chunk：** [PLAN_FILE_PATH] - 仅 Chunk N
    **参考的 Spec：** [SPEC_FILE_PATH]

    ## 检查内容

    | 类别 | 检查要点 |
    |----------|------------------|
    | 完整性 | TODOs、占位符、不完整的任务、缺失的步骤 |
    | Spec 对齐 | Chunk 覆盖相关的 spec 需求，无范围蔓延 |
    | 任务分解 | 任务原子化，边界清晰，步骤可执行 |
    | 任务语法 | 任务和步骤使用复选框语法（`- [ ]`） |
    | Chunk 大小 | 每个 chunk 少于 1000 行 |

    ## 关键检查

    特别注意：
    - 任何 TODO 标记或占位符文本
    - 说"与 X 类似"但没有实际内容的步骤
    - 不完整的任务定义
    - 缺失的验证步骤或预期输出

    ## 输出格式

    ## 计划评审 - Chunk N

    **状态：** ✅ 通过 | ❌ 发现问题

    **问题（如有）：**
    - [任务 X, 步骤 Y]: [具体问题] - [为什么重要]

    **建议（参考）：**
    - [不阻塞通过的建议]
```

**评审器返回：** 状态、问题（如有）、建议
```

- [ ] **Step 2:** 验证文件已创建

运行：`cat skills/writing-plans/plan-document-reviewer-prompt.md | head -20`
预期：显示标题和目的部分

- [ ] **Step 3:** 提交

```bash
git add skills/writing-plans/plan-document-reviewer-prompt.md
git commit -m "feat: add plan document reviewer prompt template"
```

---

### Task 4: 将评审循环添加到 Writing-Plans 技能

**文件：**
- 修改：`skills/writing-plans/SKILL.md`

- [ ] **Step 1:** 读取当前技能文件

运行：`cat skills/writing-plans/SKILL.md`

- [ ] **Step 2:** 添加逐 chunk 评审部分

在"执行交接"部分之前添加：

```markdown
## 计划评审循环

完成计划的每个 chunk 后：

1. 为当前 chunk 调度 plan-document-reviewer subagent
   - 提供：chunk 内容，spec 文档路径
2. 如果 ❌ 发现问题：
   - 在 chunk 中修复问题
   - 为该 chunk 重新调度评审器
   - 重复直到 ✅ 通过
3. 如果 ✅ 通过：继续下一个 chunk（如果是最后一个 chunk，则进行执行交接）

**Chunk 边界：** 使用 `## Chunk N: <name>` 标题来划分 chunk。每个 chunk 应 ≤1000 行且逻辑上独立。
```

- [ ] **Step 3:** 更新任务语法示例以使用复选框

将任务结构部分更改为显示复选框语法：

```markdown
### Task N: [组件名称]

- [ ] **Step 1:** 编写失败的测试
  - 文件：`tests/path/test.py`
  ...
```

- [ ] **Step 4:** 验证评审循环部分已添加

运行：`grep -A 15 "计划评审循环" skills/writing-plans/SKILL.md`
预期：显示新的评审循环部分

- [ ] **Step 5:** 验证任务语法示例已更新

运行：`grep -A 5 "Task N:" skills/writing-plans/SKILL.md`
预期：显示复选框语法 `### Task N:`

- [ ] **Step 6:** 提交

```bash
git add skills/writing-plans/SKILL.md
git commit -m "feat: add plan review loop and checkbox syntax to writing-plans skill"
```

---

## Chunk 3: 更新计划文档头部

此 chunk 更新计划文档头部模板以引用新的复选框语法要求。

### Task 5: 更新 Writing-Plans 技能中的计划头部模板

**文件：**
- 修改：`skills/writing-plans/SKILL.md`

- [ ] **Step 1:** 读取当前计划头部模板

运行：`grep -A 20 "Plan Document Header" skills/writing-plans/SKILL.md`

- [ ] **Step 2:** 更新头部模板以引用复选框语法

计划头部应注明任务和步骤使用复选框语法。更新头部注释：

```markdown
> **对于 agentic workers：** 必需：使用 superpowers:subagent-driven-development（如果 subagent 可用）或 superpowers:executing-plans 来实现此计划。任务和步骤使用复选框（`- [ ]`）语法进行跟踪。
```

- [ ] **Step 3:** 验证更改

运行：`grep -A 5 "对于 agentic workers:" skills/writing-plans/SKILL.md`
预期：显示更新后的头部，包含复选框语法说明

- [ ] **Step 4:** 提交

```bash
git add skills/writing-plans/SKILL.md
git commit -m "docs: update plan header to reference checkbox syntax"
```
