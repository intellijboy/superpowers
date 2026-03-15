# Superpowers

Superpowers 是一套为编码代理打造的完整软件开发工作流，建立在可组合的"技能（skills）"集合之上，并通过初始指令确保你的代理能够正确使用它们。

## 工作原理

从你启动编码代理的那一刻开始。当它发现你正在构建某些东西时，它*不会*直接跳进去写代码。相反，它会停下来问你真正想要做什么。

一旦它从对话中梳理出规格说明，它会以足够简短的片段展示给你，让你能够真正阅读和消化。

在你确认设计后，你的代理会制定一个足够清晰的实现计划，即使是一个热情但缺乏判断力、没有项目背景且厌恶测试的初级工程师也能遵循。它强调真正的红/绿 TDD、YAGNI（你不会需要它）和 DRY（不要重复自己）。

接下来，当你说"开始"时，它会启动一个 *subagent-driven-development*（子代理驱动开发）流程，让代理完成每个工程任务，检查和审查它们的工作，然后继续前进。Claude 通常能够自主工作数小时而不会偏离你们共同制定的计划。

还有更多内容，但这是系统的核心。由于技能会自动触发，你不需要做任何特别的事情。你的编码代理就拥有了 Superpowers。


## 赞助

如果 Superpowers 帮助你完成了一些能赚钱的事情，而且你有这个意愿，我非常感谢你能考虑[赞助我的开源工作](https://github.com/sponsors/obra)。

谢谢！

- Jesse


## 安装

**注意：** 安装方式因平台而异。Claude Code 或 Cursor 有内置的插件市场。Codex 和 OpenCode 需要手动设置。

### Claude Code 官方市场

Superpowers 可通过[官方 Claude 插件市场](https://claude.com/plugins/superpowers)获取

从 Claude 市场安装插件：

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code（通过插件市场）

在 Claude Code 中，首先注册市场：

```bash
/plugin marketplace add obra/superpowers-marketplace
```

然后从该市场安装插件：

```bash
/plugin install superpowers@superpowers-marketplace
```

### Cursor（通过插件市场）

在 Cursor Agent 聊天中，从市场安装：

```text
/add-plugin superpowers
```

或在插件市场中搜索"superpowers"。

### Codex

告诉 Codex：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

**详细文档：** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

告诉 OpenCode：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**详细文档：** [docs/README.opencode.md](docs/README.opencode.md)

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

更新：

```bash
gemini extensions update superpowers
```

### 验证安装

在你选择的平台中开始一个新会话，并请求一些应该触发技能的内容（例如，"帮我规划这个功能"或"让我们调试这个问题"）。代理应该自动调用相关的 superpowers 技能。

## 基本工作流

1. **brainstorming**（头脑风暴） - 在编写代码前激活。通过问题提炼粗略想法，探索替代方案，分段展示设计以供验证。保存设计文档。

2. **using-git-worktrees**（使用 git worktrees） - 在设计批准后激活。在新分支上创建隔离的工作空间，运行项目设置，验证干净的测试基准线。

3. **writing-plans**（编写计划） - 在设计批准后激活。将工作分解为小任务（每个2-5分钟）。每个任务都有确切的文件路径、完整的代码和验证步骤。

4. **subagent-driven-development**（子代理驱动开发）或 **executing-plans**（执行计划） - 在有计划时激活。为每个任务派遣新的子代理，进行两阶段审查（规格合规性，然后是代码质量），或带有人工检查点批量执行。

5. **test-driven-development**（测试驱动开发） - 在实现期间激活。强制执行 RED-GREEN-REFACTOR：编写失败的测试，观看它失败，编写最小代码，观看它通过，提交。删除在测试之前编写的代码。

6. **requesting-code-review**（请求代码审查） - 在任务之间激活。根据计划审查，按严重程度报告问题。关键问题会阻止进度。

7. **finishing-a-development-branch**（完成开发分支） - 在任务完成时激活。验证测试，展示选项（merge/PR/keep/discard），清理 worktree。

**代理在任何任务之前会检查相关技能。** 这是强制性的工作流程，而不是建议。

## 包含内容

### 技能库

**测试**
- **test-driven-development** - RED-GREEN-REFACTOR 循环（包含测试反模式参考）

**调试**
- **systematic-debugging** - 4阶段根本原因流程（包含 root-cause-tracing、defense-in-depth、condition-based-waiting 技术）
- **verification-before-completion** - 确保问题真正被修复

**协作**
- **brainstorming** - 苏格拉底式设计提炼
- **writing-plans** - 详细的实现计划
- **executing-plans** - 带检查点的批量执行
- **dispatching-parallel-agents** - 并发子代理工作流
- **requesting-code-review** - 预审查检查清单
- **receiving-code-review** - 响应反馈
- **using-git-worktrees** - 并行开发分支
- **finishing-a-development-branch** - 合并/PR 决策工作流
- **subagent-driven-development** - 带两阶段审查的快速迭代（规格合规性，然后是代码质量）

**元技能**
- **writing-skills** - 按照最佳实践创建新技能（包含测试方法论）
- **using-superpowers** - 技能系统介绍

## 理念

- **测试驱动开发** - 始终先写测试
- **系统性优于临时性** - 流程优于猜测
- **降低复杂度** - 简洁作为首要目标
- **证据优于声明** - 在宣布成功之前先验证

更多阅读：[Superpowers for Claude Code](https://blog.fsck.com/2025/10/09/superpowers/)

## 贡献

技能直接存放在这个仓库中。要贡献：

1. Fork 这个仓库
2. 为你的技能创建一个分支
3. 按照 `writing-skills` 技能创建和测试新技能
4. 提交 PR

完整指南请参阅 `skills/writing-skills/SKILL.md`。

## 更新

当你更新插件时，技能会自动更新：

```bash
/plugin update superpowers
```

## 许可证

MIT 许可证 - 详情请参阅 LICENSE 文件

## 支持

- **问题反馈**: https://github.com/obra/superpowers/issues
- **市场**: https://github.com/obra/superpowers-marketplace
