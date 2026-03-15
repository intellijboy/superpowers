# Superpowers 发布说明

## v5.0.2 (2026-03-11)

### 零依赖的 Brainstorm 服务器

**移除了所有内嵌的 node_modules — server.js 现在完全自包含**

- 将 Express/Chokidar/WebSocket 依赖替换为零依赖的 Node.js 服务器，使用内置的 `http`、`fs` 和 `crypto` 模块
- 移除了约 1,200 行内嵌的 `node_modules/`、`package.json` 和 `package-lock.json`
- 自定义 WebSocket 协议实现（RFC 6455 帧、ping/pong、正确的关闭握手）
- 原生 `fs.watch()` 文件监视取代了 Chokidar
- 完整的测试套件：HTTP 服务、WebSocket 协议、文件监视和集成测试

### Brainstorm 服务器可靠性

- **空闲 30 分钟后自动退出** — 当没有客户端连接时服务器关闭，防止孤儿进程
- **所有者进程跟踪** — 服务器监控父 harness PID，当拥有会话终止时退出
- **存活检查** — 技能在重用现有实例之前验证服务器是否响应
- **编码修复** — 在服务的 HTML 页面上添加正确的 `<meta charset="utf-8">`

### 子代理上下文隔离

- 所有委托技能（brainstorming、dispatching-parallel-agents、requesting-code-review、subagent-driven-development、writing-plans）现在都包含上下文隔离原则
- 子代理仅接收它们需要的上下文，防止上下文窗口污染

## v5.0.1 (2026-03-10)

### Agentskills 合规性

**Brainstorm-server 移至技能目录**

- 按照 [agentskills.io](https://agentskills.io) 规范，将 `lib/brainstorm-server/` 移至 `skills/brainstorming/scripts/`
- 所有 `${CLAUDE_PLUGIN_ROOT}/lib/brainstorm-server/` 引用替换为相对 `scripts/` 路径
- 技能现在可以跨平台完全移植 — 不需要平台特定的环境变量来定位脚本
- `lib/` 目录已移除（曾是最后剩余的内容）

### 新功能

**Gemini CLI 扩展**

- 通过仓库根目录的 `gemini-extension.json` 和 `GEMINI.md` 提供原生 Gemini CLI 扩展支持
- `GEMINI.md` 在会话开始时 @import `using-superpowers` 技能和工具映射表
- Gemini CLI 工具映射参考（`skills/using-superpowers/references/gemini-tools.md`）— 将 Claude Code 工具名称（Read、Write、Edit、Bash 等）翻译为 Gemini CLI 等效项（read_file、write_file、replace 等）
- 文档说明 Gemini CLI 限制：无子代理支持，技能回退到 `executing-plans`
- 扩展根目录位于仓库根目录以实现跨平台兼容性（避免 Windows 符号链接问题）
- README 中添加了安装说明

### 改进

**多平台 brainstorm 服务器启动**

- visual-companion.md 中的每个平台启动说明：Claude Code（默认模式）、Codex（通过 `CODEX_CI` 自动前台）、Gemini CLI（`--foreground` 与 `is_background`）以及其他环境的回退
- 服务器现在将启动 JSON 写入 `$SCREEN_DIR/.server-info`，这样即使 stdout 被后台执行隐藏，代理也能找到 URL 和端口

**Brainstorm 服务器依赖打包**

- `node_modules` 内嵌到仓库中，这样 brainstorm 服务器在全新插件安装后立即可用，无需在运行时安装 `npm`
- 从打包的依赖中移除了 `fsevents`（仅 macOS 的原生二进制文件；chokidar 在没有它的情况下也能正常回退）
- 如果 `node_modules` 丢失，通过 `npm install` 回退自动安装

**OpenCode 工具映射修复**

- `TodoWrite` → `todowrite`（之前错误映射为 `update_plan`）；已对照 OpenCode 源代码验证

### 错误修复

**Windows/Linux：单引号破坏 SessionStart hook** (#577, #529, #644, PR #585)

- hooks.json 中 `${CLAUDE_PLUGIN_ROOT}` 周围的单引号在 Windows 上失败（cmd.exe 不识别单引号为路径分隔符），在 Linux 上也失败（单引号阻止变量展开）
- 修复：用转义的双引号替换单引号 — 适用于 macOS bash、Windows cmd.exe、Windows Git Bash 和 Linux，无论路径中是否有空格
- 已在 Windows 11 (NT 10.0.26200.0) 上使用 Claude Code 2.1.72 和 Git for Windows 验证

**Brainstorming 规格审查循环被跳过** (#677)

- 规格审查循环（派遣 spec-document-reviewer 子代理，迭代直到批准）存在于"设计后"散文部分，但在检查清单和流程图中缺失
- 由于代理更可靠地遵循图表和检查清单而非散文，规格审查步骤被完全跳过
- 在检查清单中添加了步骤 7（规格审查循环），并在 dot 图中添加了相应节点
- 使用 `claude --plugin-dir` 和 `claude-session-driver` 测试：worker 现在正确派遣审查器

**Cursor 安装命令** (PR #676)

- 修复了 README 中的 Cursor 安装命令：`/plugin-add` → `/add-plugin`（通过 Cursor 2.5 发布公告确认）

**Brainstorming 中的用户审查关口** (#565)

- 在规格完成和 writing-plans 交接之间添加了明确的用户审查步骤
- 用户必须在开始实现规划之前批准规格
- 检查清单、流程图和散文都用新的关口更新了

**Session-start hook 每个平台只发出一次上下文**

- Hook 现在检测它是在 Claude Code 还是其他平台运行
- 为 Claude Code 发出 `hookSpecificOutput`，为其他平台发出 `additional_context` — 防止双重上下文注入

**Token 分析脚本中的 Linting 修复**

- `tests/claude-code/analyze-token-usage.py` 中的 `except:` → `except Exception:`

### 维护

**移除死代码**

- 删除了 `lib/skills-core.js` 及其测试（`tests/opencode/test-skills-core.js`）— 自 2026 年 2 月以来未使用
- 从 `tests/opencode/test-plugin-loading.sh` 中移除了 skills-core 存在检查

### 社区

- @karuturi — Claude Code 官方市场安装说明 (PR #610)
- @mvanhorn — session-start hook 双重发出修复，OpenCode 工具映射修复
- @daniel-graham — 裸 except 的 linting 修复
- PR #585 作者 — Windows/Linux hooks 引号修复

---

## v5.0.0 (2026-03-09)

### 重大变更

**规格和计划目录重构**

- 规格（brainstorming 输出）现在保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- 计划（writing-plans 输出）现在保存到 `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- 用户对规格/计划位置的偏好覆盖这些默认值
- 所有内部技能引用、测试文件和示例路径已更新以匹配
- 迁移：如果需要，将现有文件从 `docs/plans/` 移动到新位置

**在支持的 harness 上强制使用子代理驱动开发**

Writing-plans 不再在 subagent-driven 和 executing-plans 之间提供选择。在支持子代理的 harness（Claude Code、Codex）上，subagent-driven-development 是必需的。Executing-plans 保留给没有子代理能力的 harness，现在会告诉用户 Superpowers 在支持子代理的平台上工作得更好。

**Executing-plans 不再批量执行**

移除了"执行 3 个任务然后停止审查"的模式。计划现在连续执行，仅在遇到阻塞时停止。

**斜杠命令已弃用**

`/brainstorm`、`/write-plan` 和 `/execute-plan` 现在显示弃用通知，引导用户使用相应的技能。命令将在下一个主要版本中移除。

### 新功能

**可视化头脑风暴伴侣**

用于头脑风暴会话的可选基于浏览器的伴侣。当主题受益于可视化时，brainstorming 技能提议在浏览器窗口中与终端对话并排显示模型、图表、比较和其他内容。

- `lib/brainstorm-server/` — WebSocket 服务器，包含浏览器助手库、会话管理脚本和深色/浅色主题框架模板（"Superpowers Brainstorming" 带有 GitHub 链接）
- `skills/brainstorming/visual-companion.md` — 服务器工作流、屏幕创作和反馈收集的渐进式披露指南
- Brainstorming 技能在其流程中添加了可视化伴侣决策点：在探索项目上下文后，技能评估即将到来的问题是否涉及可视化内容，并在单独的消息中提供伴侣
- 每个问题的决策：即使在接受后，也会针对每个问题评估浏览器或终端哪个更合适
- `tests/brainstorm-server/` 中的集成测试

**文档审查系统**

使用子代理派遣对规格和计划文档的自动审查循环：

- `skills/brainstorming/spec-document-reviewer-prompt.md` — 审查器检查完整性、一致性、架构和 YAGNI
- `skills/writing-plans/plan-document-reviewer-prompt.md` — 审查器检查规格对齐、任务分解、文件结构和文件大小
- Brainstorming 在编写设计文档后派遣规格审查器
- Writing-plans 在每个部分后包含基于块的计划审查循环
- 审查循环重复直到批准或在 5 次迭代后升级
- `tests/claude-code/test-document-review-system.sh` 中的端到端测试
- `docs/superpowers/` 中的设计规格和实现计划

**跨技能管道的架构指导**

为隔离设计和文件大小感知指导添加到 brainstorming、writing-plans 和 subagent-driven-development：

- **Brainstorming** — 新部分："为隔离和清晰而设计"（清晰的边界、定义良好的接口、可独立测试的单元）和"在现有代码库中工作"（遵循现有模式、仅做针对性改进）
- **Writing-plans** — 新"文件结构"部分：在定义任务之前映射文件和职责。新"范围检查"后盾：捕获本应在头脑风暴期间分解的多子系统规格
- **SDD 实现者** — 新"代码组织"部分（遵循计划的文件结构、报告关于文件增长的担忧）和"当你无法应付时"升级指导
- **SDD 代码质量审查器** — 现在检查架构、单元分解、计划一致性和文件增长
- **规格/计划审查器** — 架构和文件大小添加到审查标准
- **范围评估** — Brainstorming 现在评估项目对于单个规格是否太大。多子系统请求被早期标记并分解为子项目，每个都有自己的规格 → 计划 → 实现循环

**子代理驱动开发改进**

- **模型选择** — 根据任务类型选择模型能力的指导：机械实现使用廉价模型、集成使用标准模型、架构和审查使用高性能模型
- **实现者状态协议** — 子代理现在报告 DONE、DONE_WITH_CONCERNS、BLOCKED 或 NEEDS_CONTEXT。控制器适当处理每种状态：使用更多上下文重新派遣、升级模型能力、分解任务或升级给人工

### 改进

**指令优先级层次**

在 using-superpowers 中添加了明确的优先级排序：

1. 用户的明确指令（CLAUDE.md、AGENTS.md、直接请求）— 最高优先级
2. Superpowers 技能 — 覆盖默认系统行为
3. 默认系统提示 — 最低优先级

如果 CLAUDE.md 或 AGENTS.md 说"不要使用 TDD"而技能说"始终使用 TDD"，用户的指令优先。

**SUBAGENT-STOP 关口**

在 using-superpowers 中添加了 `<SUBAGENT-STOP>` 块。为特定任务派遣的子代理现在跳过技能检查，而不是激活 1% 规则并调用完整的技能工作流。

**多平台改进**

- Codex 工具映射移至渐进式披露参考文件（`references/codex-tools.md`）
- 添加了平台适配指针，以便非 Claude Code 平台可以找到工具等效项
- 计划头部现在称呼"代理工作者"而不是专门称呼"Claude"
- `docs/README.codex.md` 中记录了协作功能要求

**Writing-plans 模板更新**

- 计划步骤现在使用复选框语法（`- [ ] **步骤 N:**`）进行进度跟踪
- 计划头部现在引用 subagent-driven-development 和 executing-plans，并带有平台感知路由

---

## v4.3.1 (2026-02-21)

### 新增

**Cursor 支持**

Superpowers 现在可与 Cursor 的插件系统一起使用。包括 `.cursor-plugin/plugin.json` 清单和 README 中特定于 Cursor 的安装说明。SessionStart hook 输出现在包含 `additional_context` 字段以及现有的 `hookSpecificOutput.additionalContext`，以实现 Cursor hook 兼容性。

### 修复

**Windows：恢复了多语言包装器以实现可靠的 hook 执行** (#518, #504, #491, #487, #466, #440)

Claude Code 在 Windows 上的 `.sh` 自动检测会在 hook 命令前添加 `bash`，导致执行失败。修复：

- 将 `session-start.sh` 重命名为 `session-start`（无扩展名），这样自动检测不会干扰
- 恢复了 `run-hook.cmd` 多语言包装器，具有多位置 bash 发现（标准 Git for Windows 路径，然后是 PATH 回退）
- 如果找不到 bash 则静默退出而不是报错
- 在 Unix 上，包装器通过 `exec bash` 直接运行脚本
- 使用 POSIX 安全的 `dirname "$0"` 路径解析（适用于 dash/sh，不仅仅是 bash）

这修复了 Windows 上路径中有空格、缺少 WSL、MSYS 上 `set -euo pipefail` 脆弱性和反斜杠损坏的 SessionStart 失败。

## v4.3.0 (2026-02-12)

此修复应该大大提高 superpowers 技能的合规性，并减少 Claude 无意中进入其原生计划模式的机会。

### 变更

**Brainstorming 技能现在强制执行其工作流而不是描述它**

模型正在跳过设计阶段并直接跳到实现技能如 frontend-design，或者将整个头脑风暴过程折叠成单个文本块。该技能现在使用硬关口、强制性检查清单和 graphviz 流程图来强制合规：

- `<HARD-GATE>`：在展示设计并获得用户批准之前，不能使用实现技能、代码或脚手架
- 必须作为任务创建并按顺序完成的明确检查清单（6 项）
- Graphviz 流程图，`writing-plans` 作为唯一有效的终端状态
- 针对"这太简单了不需要设计"的反模式标注 — 模型用来跳过流程的确切合理化
- 基于部分复杂度而非项目复杂度的设计部分大小调整

**Using-superpowers 工作流图拦截 EnterPlanMode**

在技能流程图中添加了 `EnterPlanMode` 拦截。当模型即将进入 Claude 的原生计划模式时，它检查头脑风暴是否已经发生，并通过头脑风暴技能路由。计划模式永远不会进入。

### 修复

**SessionStart hook 现在同步运行**

将 hooks.json 中的 `async: true` 改为 `async: false`。当异步时，hook 可能无法在模型第一轮之前完成，意味着 using-superpowers 指令不在第一条消息的上下文中。

## v4.2.0 (2026-02-05)

### 重大变更

**Codex：用原生技能发现替换引导 CLI**

`superpowers-codex` 引导 CLI、Windows `.cmd` 包装器和相关的引导内容文件已被移除。Codex 现在通过 `~/.agents/skills/superpowers/` 符号链接使用原生技能发现，因此不再需要旧的 `use_skill`/`find_skills` CLI 工具。

安装现在只是克隆 + 符号链接（在 INSTALL.md 中有文档）。不需要 Node.js 依赖。旧的 `~/.codex/skills/` 路径已弃用。

### 修复

**Windows：修复了 Claude Code 2.1.x hook 执行** (#331)

Claude Code 2.1.x 更改了 hooks 在 Windows 上的执行方式：它现在自动检测命令中的 `.sh` 文件并添加 `bash` 前缀。这破坏了多语言包装器模式，因为 `bash "run-hook.cmd" session-start.sh` 尝试将 `.cmd` 文件作为 bash 脚本执行。

修复：hooks.json 现在直接调用 session-start.sh。Claude Code 2.1.x 自动处理 bash 调用。还添加了 .gitattributes 以强制 shell 脚本使用 LF 行尾（修复 Windows 检出时的 CRLF 问题）。

**Windows：SessionStart hook 异步运行以防止终端冻结** (#404, #413, #414, #419)

同步的 SessionStart hook 阻止 TUI 在 Windows 上进入原始模式，冻结所有键盘输入。异步运行 hook 可以防止冻结，同时仍然注入 superpowers 上下文。

**Windows：修复了 O(n^2) 的 `escape_for_json` 性能**

使用 `${input:$i:1}` 的逐字符循环在 bash 中是 O(n^2)，由于子字符串复制开销。在 Windows Git Bash 上这需要 60+ 秒。用 bash 参数替换（`${s//old/new}`）替换，它将每个模式作为单个 C 级传递运行 — 在 macOS 上快 7 倍，在 Windows 上大幅更快。

**Codex：修复了 Windows/PowerShell 调用** (#285, #243)

- Windows 不遵循 shebang，因此直接调用无扩展名的 `superpowers-codex` 脚本会触发"打开方式"对话框。所有调用现在都添加 `node` 前缀。
- 修复了 Windows 上的 `~/` 路径展开 — PowerShell 在作为参数传递给 `node` 时不展开 `~`。改为 `$HOME`，在 bash 和 PowerShell 中都能正确展开。

**Codex：修复了安装程序中的路径解析**

使用 `fileURLToPath()` 而不是手动 URL 路径名解析，以正确处理所有平台上包含空格和特殊字符的路径。

**Codex：修复了 writing-skills 中过时的技能路径**

更新了 `~/.codex/skills/` 引用（已弃用）为 `~/.agents/skills/` 以进行原生发现。

### 改进

**实现前现在要求 Worktree 隔离**

为 `subagent-driven-development` 和 `executing-plans` 添加了 `using-git-worktrees` 作为必需技能。实现工作流现在明确要求在开始工作之前设置隔离的 worktree，防止直接在 main 上意外工作。

**主分支保护软化为需要明确同意**

技能现在不再完全禁止在主分支上工作，而是允许在用户明确同意的情况下进行。更灵活，同时仍然确保用户意识到影响。

**简化了安装验证**

从验证步骤中移除了 `/help` 命令检查和特定的斜杠命令列表。技能主要通过描述你想做什么来调用，而不是通过运行特定命令。

**Codex：在引导中澄清了子代理工具映射**

改进了 Codex 工具如何映射到 Claude Code 等效项以进行子代理工作流的文档。

### 测试

- 添加了 subagent-driven-development 的 worktree 要求测试
- 添加了主分支红旗警告测试
- 修复了技能识别测试断言中的大小写敏感性

---

## v4.1.1 (2026-01-23)

### 修复

**OpenCode：按照官方文档标准化为 `plugins/` 目录** (#343)

OpenCode 的官方文档使用 `~/.config/opencode/plugins/`（复数）。我们的文档之前使用 `plugin/`（单数）。虽然 OpenCode 接受两种形式，但我们已标准化为官方约定以避免混淆。

变更：
- 在仓库结构中将 `.opencode/plugin/` 重命名为 `.opencode/plugins/`
- 更新了所有安装文档（INSTALL.md、README.opencode.md）跨所有平台
- 更新了测试脚本以匹配

**OpenCode：修复了符号链接说明** (#339, #342)

- 在 `ln -s` 之前添加了明确的 `rm`（修复重新安装时的"文件已存在"错误）
- 添加了 INSTALL.md 中缺失的技能符号链接步骤
- 从已弃用的 `use_skill`/`find_skills` 更新为原生 `skill` 工具引用

---

## v4.1.0 (2026-01-23)

### 重大变更

**OpenCode：切换到原生技能系统**

Superpowers for OpenCode 现在使用 OpenCode 的原生 `skill` 工具，而不是自定义的 `use_skill`/`find_skills` 工具。这是一个更干净的集成，与 OpenCode 的内置技能发现一起工作。

**需要迁移：** 技能必须符号链接到 `~/.config/opencode/skills/superpowers/`（参见更新的安装文档）。

### 修复

**OpenCode：修复了会话开始时的代理重置** (#226)

之前使用 `session.prompt({ noReply: true })` 的引导注入方法导致 OpenCode 在第一条消息时将选定的代理重置为"build"。现在使用 `experimental.chat.system.transform` hook，它直接修改系统提示而没有副作用。

**OpenCode：修复了 Windows 安装** (#232)

- 移除了对 `skills-core.js` 的依赖（当文件被复制而不是符号链接时消除了损坏的相对导入）
- 为 cmd.exe、PowerShell 和 Git Bash 添加了全面的 Windows 安装文档
- 为每个平台记录了正确的符号链接与连接使用

**Claude Code：修复了 Claude Code 2.1.x 的 Windows hook 执行**

Claude Code 2.1.x 更改了 hooks 在 Windows 上的执行方式：它现在自动检测 `.sh` 文件在命令中并添加 `bash` 前缀。这破坏了多语言包装器模式，因为 `bash "run-hook.cmd" session-start.sh` 尝试将 .cmd 文件作为 bash 脚本执行。

修复：hooks.json 现在直接调用 session-start.sh。Claude Code 2.1.x 自动处理 bash 调用。还添加了 .gitattributes 以强制 shell 脚本使用 LF 行尾（修复 Windows 检出时的 CRLF 问题）。

---

## v4.0.3 (2025-12-26)

### 改进

**加强了 using-superpowers 技能以处理明确的技能请求**

解决了即使当用户明确按名称请求技能（例如"subagent-driven-development, please"）时 Claude 也会跳过调用技能的失败模式。Claude 会认为"我知道那是什么意思"并直接开始工作，而不是加载技能。

变更：
- 更新"规则"为"调用相关或请求的技能"而不是"检查技能" — 强调主动调用而非被动检查
- 添加了"在任何响应或操作之前" — 原来的措辞只提到"响应"，但 Claude 有时会先采取行动而不响应
- 添加了保证调用错误的技能也没关系 — 减少犹豫
- 添加了新的红旗："我知道那是什么意思" → 知道概念 ≠ 使用技能

**添加了明确的技能请求测试**

`tests/explicit-skill-requests/` 中的新测试套件，验证当用户按名称请求技能时 Claude 正确调用技能。包括单轮和多轮测试场景。

## v4.0.2 (2025-12-23)

### 修复

**斜杠命令现在仅限用户使用**

为所有三个斜杠命令（`/brainstorm`、`/execute-plan`、`/write-plan`）添加了 `disable-model-invocation: true`。Claude 无法再通过 Skill 工具调用这些命令 — 它们仅限于手动用户调用。

底层技能（`superpowers:brainstorming`、`superpowers:executing-plans`、`superpowers:writing-plans`）仍然可供 Claude 自主调用。此更改防止当 Claude 调用只是重定向到技能的命令时的混淆。

## v4.0.1 (2025-12-23)

### 修复

**澄清了如何在 Claude Code 中访问技能**

修复了一个令人困惑的模式，即 Claude 会通过 Skill 工具调用技能，然后尝试单独读取技能文件。`using-superpowers` 技能现在明确指出 Skill 工具直接加载技能内容 — 不需要读取文件。

- 在 `using-superpowers` 中添加了"如何访问技能"部分
- 在指令中将"读取技能"改为"调用技能"
- 更新了斜杠命令以使用完全限定的技能名称（例如 `superpowers:brainstorming`）

**在 receiving-code-review 中添加了 GitHub 线程回复指导** (h/t @ralphbean)

添加了关于在原始线程中回复内联审查注释而不是作为顶级 PR 注释的说明。

**在 writing-skills 中添加了自动化优于文档的指导** (h/t @EthanJStark)

添加了指导，即机械约束应该自动化，而不是记录 — 将技能保留用于判断调用。

## v4.0.0 (2025-12-17)

### 新功能

**subagent-driven-development 中的两阶段代码审查**

子代理工作流现在在每个任务后使用两个单独的审查阶段：

1. **规格合规性审查** — 怀疑的审查器验证实现是否完全匹配规格。捕获缺失的需求和过度构建。不会信任实现者的报告 — 读取实际代码。

2. **代码质量审查** — 仅在规格合规性通过后运行。审查代码整洁度、测试覆盖率、可维护性。

这捕获了代码写得好但不匹配请求内容的常见失败模式。审查是循环，不是一次性的：如果审查器发现问题，实现者修复它们，然后审查器再次检查。

其他子代理工作流改进：
- 控制器向工作者提供完整的任务文本（而不是文件引用）
- 工作者可以在工作之前和工作期间提出澄清问题
- 报告完成前的自我审查检查清单
- 开始时读取一次计划，提取到 TodoWrite

`skills/subagent-driven-development/` 中的新提示模板：
- `implementer-prompt.md` — 包括自我审查检查清单，鼓励提问
- `spec-reviewer-prompt.md` — 对需求的怀疑验证
- `code-quality-reviewer-prompt.md` — 标准代码审查

**调试技术与工具整合**

`systematic-debugging` 现在捆绑了支持技术和工具：
- `root-cause-tracing.md` — 通过调用栈向后跟踪 bug
- `defense-in-depth.md` — 在多层添加验证
- `condition-based-waiting.md` — 用条件轮询替换任意超时
- `find-polluter.sh` — 用于查找哪个测试造成污染的二分脚本
- `condition-based-waiting-example.ts` — 来自真实调试会话的完整实现

**测试反模式参考**

`test-driven-development` 现在包括 `testing-anti-patterns.md`，涵盖：
- 测试模拟行为而不是真实行为
- 向生产类添加仅测试的方法
- 在不了解依赖的情况下模拟
- 隐藏结构假设的不完整模拟

**技能测试基础设施**

三个新的测试框架用于验证技能行为：

`tests/skill-triggering/` — 验证技能从天真提示触发而无需明确命名。测试 6 个技能以确保描述单独足够。

`tests/claude-code/` — 使用 `claude -p` 进行无头测试的集成测试。通过会话转录（JSONL）分析验证技能使用。包括用于成本跟踪的 `analyze-token-usage.py`。

`tests/subagent-driven-dev/` — 具有两个完整测试项目的端到端工作流验证：
- `go-fractals/` — 带有 Sierpinski/Mandelbrot 的 CLI 工具（10 个任务）
- `svelte-todo/` — 带有 localStorage 和 Playwright 的 CRUD 应用（12 个任务）

### 主要变更

**DOT 流程图作为可执行规范**

使用 DOT/GraphViz 流程图作为权威流程定义重写了关键技能。散文成为支持内容。

**描述陷阱**（在 `writing-skills` 中记录）：发现当描述包含工作流摘要时，技能描述会覆盖流程图内容。Claude 遵循简短的描述而不是阅读详细的流程图。修复：描述必须是仅触发（"当 X 时使用"），没有流程细节。

**using-superpowers 中的技能优先级**

当多个技能适用时，流程技能（brainstorming、debugging）现在明确优先于实现技能。"构建 X"首先触发头脑风暴，然后是领域技能。

**brainstorming 触发加强**

描述改为命令式："在任何创造性工作之前必须使用此技能 — 创建功能、构建组件、添加功能或修改行为。"

### 重大变更

**技能整合** — 合并了六个独立技能：
- `root-cause-tracing`、`defense-in-depth`、`condition-based-waiting` → 捆绑在 `systematic-debugging/` 中
- `testing-skills-with-subagents` → 捆绑在 `writing-skills/` 中
- `testing-anti-patterns` → 捆绑在 `test-driven-development/` 中
- `sharing-skills` 已移除（过时）

### 其他改进

- **render-graphs.js** — 从技能中提取 DOT 图并渲染为 SVG 的工具
- **using-superpowers 中的合理化表** — 可扫描格式，包括新条目："我首先需要更多上下文"、"让我先探索"、"这感觉很有成效"
- **docs/testing.md** — 使用 Claude Code 集成测试测试技能的指南

---

## v3.6.2 (2025-12-03)

### 修复

- **Linux 兼容性**：修复了多语言 hook 包装器（`run-hook.cmd`）以使用符合 POSIX 的语法
  - 在第 16 行用标准的 `$0` 替换了特定于 bash 的 `${BASH_SOURCE[0]:-$0}`
  - 解决了 Ubuntu/Debian 系统上的"Bad substitution"错误，其中 `/bin/sh` 是 dash
  - 修复 #141

---

## v3.5.1 (2025-11-24)

### 变更

- **OpenCode 引导重构**：从 `chat.message` hook 切换到 `session.created` 事件进行引导注入
  - 引导现在通过带有 `noReply: true` 的 `session.prompt()` 在会话创建时注入
  - 明确告诉模型 using-superpowers 已加载以防止冗余技能加载
  - 将引导内容生成整合到共享的 `getBootstrapContent()` 助手中
  - 更干净的单实现方法（移除了回退模式）

---

## v3.5.0 (2025-11-23)

### 新增

- **OpenCode 支持**：OpenCode.ai 的原生 JavaScript 插件
  - 自定义工具：`use_skill` 和 `find_skills`
  - 用于技能在上下文压缩期间持久化的消息插入模式
  - 通过 chat.message hook 的自动上下文注入
  - session.compacted 事件上的自动重新注入
  - 三层技能优先级：项目 > 个人 > superpowers
  - 项目本地技能支持（`.opencode/skills/`）
  - 与 Codex 代码复用的共享核心模块（`lib/skills-core.js`）
  - 具有适当隔离的自动化测试套件（`tests/opencode/`）
  - 特定平台的文档（`docs/README.opencode.md`、`docs/README.codex.md`）

### 变更

- **重构的 Codex 实现**：现在使用共享的 `lib/skills-core.js` ES 模块
  - 消除了 Codex 和 OpenCode 之间的代码重复
  - 技能发现和解析的单一事实来源
  - Codex 通过 Node.js 互操作成功加载 ES 模块

- **改进的文档**：重写了 README 以清楚地解释问题/解决方案
  - 移除了重复部分和冲突信息
  - 添加了完整的工作流描述（brainstorm → plan → execute → finish）
  - 简化了平台安装说明
  - 强调技能检查协议而非自动激活声明

---

## v3.4.1 (2025-10-31)

### 改进

- 优化了 superpowers 引导以消除冗余的技能执行。`using-superpowers` 技能内容现在直接在会话上下文中提供，并有明确的指导仅对其他技能使用 Skill 工具。这减少了开销，并防止了令人困惑的循环，即代理尽管已经从会话开始获得内容，但仍会手动执行 `using-superpowers`。

## v3.4.0 (2025-10-30)

### 改进

- 简化了 `brainstorming` 技能以回归原始的对话愿景。移除了具有正式检查清单的重型 6 阶段流程，转而支持自然对话：一次问一个问题，然后以 200-300 字的部分呈现设计并进行验证。保留文档和实现交接功能。

## v3.3.1 (2025-10-28)

### 改进

- 更新了 `brainstorming` 技能以在提问前要求自主侦察，鼓励推荐驱动的决策，并防止代理将优先级委派回给人工。
- 按照 Strunk 的"风格要素"原则（省略不必要的词、将否定形式转换为肯定形式、改进平行结构）对 `brainstorming` 技能应用了写作清晰度改进。

### 错误修复

- 澄清了 `writing-skills` 指导，使其指向正确的代理特定个人技能目录（Claude Code 的 `~/.claude/skills`，Codex 的 `~/.codex/skills`）。

## v3.3.0 (2025-10-28)

### 新功能

**实验性 Codex 支持**
- 添加了具有 bootstrap/use-skill/find-skills 命令的统一 `superpowers-codex` 脚本
- 跨平台 Node.js 实现（适用于 Windows、macOS、Linux）
- 命名空间技能：superpowers 技能为 `superpowers:skill-name`，个人技能为 `skill-name`
- 名称匹配时个人技能覆盖 superpowers 技能
- 干净的技能显示：显示名称/描述而没有原始 frontmatter
- 有用的上下文：显示每个技能的支持文件目录
- Codex 的工具映射：TodoWrite→update_plan、子代理→手动回退等
- 具有最小 AGENTS.md 的引导集成，用于自动启动
- 特定于 Codex 的完整安装指南和引导说明

**与 Claude Code 集成的关键区别：**
- 单一统一脚本而不是单独的工具
- Codex 特定等效项的工具替换系统
- 简化的子代理处理（手动工作而不是委派）
- 更新的术语："Superpowers 技能"而不是"核心技能"

### 添加的文件
- `.codex/INSTALL.md` — Codex 用户的安装指南
- `.codex/superpowers-bootstrap.md` — 带有 Codex 适配的引导说明
- `.codex/superpowers-codex` — 具有所有功能的统一 Node.js 可执行文件

**注意：** Codex 支持是实验性的。该集成提供核心 superpowers 功能，但可能需要根据用户反馈进行改进。

## v3.2.3 (2025-10-23)

### 改进

**更新了 using-superpowers 技能以使用 Skill 工具而不是 Read 工具**
- 将技能调用指令从 Read 工具更改为 Skill 工具
- 更新了描述："使用 Read 工具" → "使用 Skill 工具"
- 更新了步骤 3："使用 Read 工具" → "使用 Skill 工具读取并运行"
- 更新了合理化列表："读取当前版本" → "运行当前版本"

Skill 工具是在 Claude Code 中调用技能的正确机制。此更新更正了引导指令以引导代理使用正确的工具。

### 变更的文件
- 更新：`skills/using-superpowers/SKILL.md` — 将工具引用从 Read 更改为 Skill

## v3.2.2 (2025-10-21)

### 改进

**加强了 using-superpowers 技能以防止代理合理化**
- 添加了具有关于强制性技能检查绝对语言的 EXTREMELY-IMPORTANT 块
  - "如果有 1% 的可能性技能适用，你必须读取它"
  - "你没有选择。你不能合理化你的出路。"
- 添加了强制性首次响应协议检查清单
  - 代理在任何响应之前必须完成的 5 步流程
  - 明确的"没有这个就响应 = 失败"后果
- 添加了常见合理化部分，包含 8 个具体的逃避模式
  - "这只是一个简单的问题" → 错误
  - "我可以快速检查文件" → 错误
  - "让我先收集信息" → 错误
  - 加上观察到的代理行为中的另外 5 个模式

这些更改解决了观察到的代理行为，尽管有明确的指令，他们仍会合理化绕过技能使用。强硬的语言和先发制人的反驳论点旨在使不合规更难。

### 变更的文件
- 更新：`skills/using-superpowers/SKILL.md` — 添加了三层强制以防止技能跳过合理化

## v3.2.1 (2025-10-20)

### 新功能

**代码审查代理现在包含在插件中**
- 在插件的 `agents/` 目录中添加了 `superpowers:code-reviewer` 代理
- 代理根据计划和编码标准提供系统的代码审查
- 以前需要用户拥有个人代理配置
- 所有技能引用更新为使用命名空间的 `superpowers:code-reviewer`
- 修复 #55

### 变更的文件
- 新增：`agents/code-reviewer.md` — 带有审查检查清单和输出格式的代理定义
- 更新：`skills/requesting-code-review/SKILL.md` — 对 `superpowers:code-reviewer` 的引用
- 更新：`skills/subagent-driven-development/SKILL.md` — 对 `superpowers:code-reviewer` 的引用

## v3.2.0 (2025-10-18)

### 新功能

**头脑风暴工作流中的设计文档**
- 在头脑风暴技能中添加了阶段 4：设计文档
- 设计文档现在在实现之前写入 `docs/plans/YYYY-MM-DD-<topic>-design.md`
- 恢复了在技能转换期间丢失的原始头脑风暴命令的功能
- 文档在 worktree 设置和实现规划之前写入
- 用子代理测试以验证在时间压力下的合规性

### 重大变更

**技能引用命名空间标准化**
- 所有内部技能引用现在使用 `superpowers:` 命名空间前缀
- 更新的格式：`superpowers:test-driven-development`（以前只是 `test-driven-development`）
- 影响所有 REQUIRED SUB-SKILL、RECOMMENDED SUB-SKILL 和 REQUIRED BACKGROUND 引用
- 与使用 Skill 工具调用技能的方式保持一致
- 更新的文件：brainstorming、executing-plans、subagent-driven-development、systematic-debugging、testing-skills-with-subagents、writing-plans、writing-skills

### 改进

**设计与实现计划命名**
- 设计文档使用 `-design.md` 后缀以防止文件名冲突
- 实现计划继续使用现有的 `YYYY-MM-DD-<feature-name>.md` 格式
- 两者都存储在 `docs/plans/` 目录中，命名清晰区分

## v3.1.1 (2025-10-17)

### 错误修复

- **修复了 README 中的命令语法** (#44) — 更新了所有命令引用以使用正确的命名空间语法（`/superpowers:brainstorm` 而不是 `/brainstorm`）。插件提供的命令由 Claude Code 自动命名空间以避免插件之间的冲突。

## v3.1.0 (2025-10-17)

### 重大变更

**技能名称标准化为小写**
- 所有技能 frontmatter `name:` 字段现在使用与目录名称匹配的小写 kebab-case
- 示例：`brainstorming`、`test-driven-development`、`using-git-worktrees`
- 所有技能公告和交叉引用更新为小写格式
- 这确保了目录名称、frontmatter 和文档之间的命名一致

### 新功能

**增强的头脑风暴技能**
- 添加了显示阶段、活动和工具使用的快速参考表
- 添加了可复制的工作流检查清单用于跟踪进度
- 添加了决策流程图用于何时 revisit 早期阶段
- 添加了带有具体示例的综合 AskUserQuestion 工具指导
- 添加了"问题模式"部分，解释何时使用结构化与开放式问题
- 将关键原则重构为可扫描的表格

**Anthropic 最佳实践集成**
- 添加了 `skills/writing-skills/anthropic-best-practices.md` — 官方 Anthropic 技能创作指南
- 在 writing-skills SKILL.md 中引用以获得综合指导
- 提供渐进式披露、工作流和评估的模式

### 改进

**技能交叉引用清晰度**
- 所有技能引用现在使用明确的要求标记：
  - `**REQUIRED BACKGROUND:**` — 你必须理解的先决条件
  - `**REQUIRED SUB-SKILL:**` — 工作流中必须使用的技能
  - `**Complementary skills:**` — 可选但有帮助的相关技能
- 移除了旧的路径格式（`skills/collaboration/X` → 只是 `X`）
- 使用分类关系（必需与补充）更新了集成部分
- 使用最佳实践更新了交叉引用文档

**与 Anthropic 最佳实践对齐**
- 修复了描述语法和语态（完全第三人称）
- 添加了用于扫描的快速参考表
- 添加了 Claude 可以复制和跟踪的工作流检查清单
- 适当使用流程图用于非明显的决策点
- 改进了可扫描的表格格式
- 所有技能都远低于 500 行建议

### 错误修复

- **重新添加了缺失的命令重定向** — 恢复了在 v3.0 迁移中意外删除的 `commands/brainstorm.md` 和 `commands/write-plan.md`
- 修复了 `defense-in-depth` 名称不匹配（曾是 `Defense-in-Depth-Validation`）
- 修复了 `receiving-code-review` 名称不匹配（曾是 `Code-Review-Reception`）
- 修复了 `commands/brainstorm.md` 引用正确的技能名称
- 移除了对不存在相关技能的引用

### 文档

**writing-skills 改进**
- 使用明确的要求标记更新了交叉引用指导
- 添加了对 Anthropic 官方最佳实践的引用
- 改进了显示正确技能引用格式的示例

## v3.0.1 (2025-10-16)

### 变更

我们现在使用 Anthropic 的第一方技能系统！

## v2.0.2 (2025-10-12)

### 错误修复

- **修复了当本地技能仓库领先于上游时的错误警告** — 初始化脚本在本地仓库有领先于上游的提交时错误地警告"上游有新技能"。逻辑现在正确区分三种 git 状态：本地落后（应更新）、本地领先（无警告）、已分叉（应警告）。

## v2.0.1 (2025-10-12)

### 错误修复

- **修复了插件上下文中的 session-start hook 执行** (#8, PR #9) — hook 静默失败并显示"Plugin hook error"，阻止技能上下文加载。通过以下方式修复：
  - 当 BASH_SOURCE 在 Claude Code 的执行上下文中未绑定时，使用 `${BASH_SOURCE[0]:-$0}` 回退
  - 添加 `|| true` 以优雅地处理过滤状态标志时的空 grep 结果

---

# Superpowers v2.0.0 发布说明

## 概述

Superpowers v2.0 通过重大的架构转变，使技能更易于访问、维护和社区驱动。

头条变化是**技能仓库分离**：所有技能、脚本和文档都已从插件移动到专用仓库（[obra/superpowers-skills](https://github.com/obra/superpowers-skills)）。这将 superpowers 从单体插件转变为管理技能仓库本地克隆的轻量级 shim。技能在会话开始时自动更新。用户通过标准 git 工作流 fork 和贡献改进。技能库独立于插件版本。

除基础设施外，此版本还添加了九个专注于问题解决、研究和架构的新技能。我们用命令式语气和更清晰的结构重写了核心 **using-skills** 文档，使 Claude 更容易理解何时以及如何使用技能。**find-skills** 现在输出你可以直接粘贴到 Read 工具中的路径，消除了技能发现工作流中的摩擦。

用户体验无缝操作：插件自动处理克隆、fork 和更新。贡献者发现新架构使改进和共享技能变得微不足道。此版本为技能作为社区资源快速发展奠定了基础。

## 重大变更

### 技能仓库分离

**最大的变化：** 技能不再存在于插件中。它们已移动到 [obra/superpowers-skills](https://github.com/obra/superpowers-skills) 的单独仓库。

**这对您意味着什么：**

- **首次安装：** 插件自动将技能克隆到 `~/.config/superpowers/skills/`
- **Fork：** 在设置期间，如果安装了 `gh`，你将被提供 fork 技能仓库的选项
- **更新：** 技能在会话开始时自动更新（可能时快进）
- **贡献：** 在分支上工作，本地提交，向上游提交 PR
- **不再有覆盖：** 旧的两层系统（个人/核心）替换为单仓库分支工作流

**迁移：**

如果你有现有安装：
1. 你的旧 `~/.config/superpowers/.git` 将备份到 `~/.config/superpowers/.git.bak`
2. 旧技能将备份到 `~/.config/superpowers/skills.bak`
3. 将在 `~/.config/superpowers/skills/` 创建 obra/superpowers-skills 的全新克隆

### 移除的功能

- **个人 superpowers 覆盖系统** — 替换为 git 分支工作流
- **setup-personal-superpowers hook** — 被 initialize-skills.sh 替换

## 新功能

### 技能仓库基础设施

**自动克隆和设置**（`lib/initialize-skills.sh`）
- 首次运行时克隆 obra/superpowers-skills
- 如果安装了 GitHub CLI，提供 fork 创建
- 正确设置 upstream/origin remotes
- 处理从旧安装的迁移

**自动更新**
- 每次会话开始时从跟踪远程获取
- 可能时使用快进自动合并
- 需要手动同步时通知（分支分叉）
- 使用 pulling-updates-from-skills-repository 技能进行手动同步

### 新技能

**问题解决技能**（`skills/problem-solving/`）
- **collision-zone-thinking** — 强制不相关的概念在一起以获得涌现的洞察
- **inversion-exercise** — 翻转假设以揭示隐藏的约束
- **meta-pattern-recognition** — 发现跨领域的通用原则
- **scale-game** — 在极端情况下测试以暴露基本真理
- **simplification-cascades** — 找到能消除多个组件的洞察
- **when-stuck** — 派遣到正确的问题解决技术

**研究技能**（`skills/research/`）
- **tracing-knowledge-lineages** — 理解思想如何随时间演变

**架构技能**（`skills/architecture/`）
- **preserving-productive-tensions** — 保留多种有效方法，而不是强制过早解决

### 技能改进

**using-skills（原名 getting-started）**
- 从 getting-started 重命名为 using-skills
- 用命令式语气完全重写（v4.0.0）
- 前置关键规则
- 为所有工作流添加了"为什么"解释
- 引用中始终包含 /SKILL.md 后缀
- 更清楚地区分刚性规则和灵活模式

**writing-skills**
- 交叉引用指导从 using-skills 移出
- 添加了 token 效率部分（字数目标）
- 改进了 CSO（Claude 搜索优化）指导

**sharing-skills**
- 为新的分支和 PR 工作流更新（v2.0.0）
- 移除了个人/核心分割引用

**pulling-updates-from-skills-repository**（新）
- 与上游同步的完整工作流
- 替换旧的"updating-skills"技能

### 工具改进

**find-skills**
- 现在输出带有 /SKILL.md 后缀的完整路径
- 使路径可直接用于 Read 工具
- 更新了帮助文本

**skill-run**
- 从 scripts/ 移动到 skills/using-skills/
- 改进了文档

### 插件基础设施

**会话开始 Hook**
- 现在从技能仓库位置加载
- 会话开始时显示完整技能列表
- 打印技能位置信息
- 显示更新状态（成功更新/落后于上游）
- 将"技能落后"警告移到输出末尾

**环境变量**
- `SUPERPOWERS_SKILLS_ROOT` 设置为 `~/.config/superpowers/skills`
- 在所有路径中一致使用

## 错误修复

- 修复了 fork 时的重复上游远程添加
- 修复了 find-skills 输出中的双重"skills/"前缀
- 从 session-start 中移除了过时的 setup-personal-superpowers 调用
- 修复了整个 hooks 和 commands 中的路径引用

## 文档

### README
- 为新的技能仓库架构更新
- 指向 superpowers-skills 仓库的突出链接
- 更新了自动更新描述
- 修复了技能名称和引用
- 更新了元技能列表

### 测试文档
- 添加了全面的测试检查清单（`docs/TESTING-CHECKLIST.md`）
- 为测试创建了本地市场配置
- 记录了手动测试场景

## 技术细节

### 文件变更

**新增：**
- `lib/initialize-skills.sh` — 技能仓库初始化和自动更新
- `docs/TESTING-CHECKLIST.md` — 手动测试场景
- `.claude-plugin/marketplace.json` — 本地测试配置

**移除：**
- `skills/` 目录（82 个文件）— 现在在 obra/superpowers-skills 中
- `scripts/` 目录 — 现在在 obra/superpowers-skills/skills/using-skills/ 中
- `hooks/setup-personal-superpowers.sh` — 已过时

**修改：**
- `hooks/session-start.sh` — 使用来自 ~/.config/superpowers/skills 的技能
- `commands/brainstorm.md` — 更新路径为 SUPERPOWERS_SKILLS_ROOT
- `commands/write-plan.md` — 更新路径为 SUPERPOWERS_SKILLS_ROOT
- `commands/execute-plan.md` — 更新路径为 SUPERPOWERS_SKILLS_ROOT
- `README.md` — 为新架构完全重写

### 提交历史

此版本包括：
- 20+ 个技能仓库分离提交
- PR #1：受 Amplifier 启发的问题解决和研究技能
- PR #2：个人 superpowers 覆盖系统（后来被替换）
- 多项技能改进和文档改进

## 升级说明

### 全新安装

```bash
# 在 Claude Code 中
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

插件自动处理所有事情。

### 从 v1.x 升级

1. **备份你的个人技能**（如果有的话）：
   ```bash
   cp -r ~/.config/superpowers/skills ~/superpowers-skills-backup
   ```

2. **更新插件：**
   ```bash
   /plugin update superpowers
   ```

3. **在下次会话开始时：**
   - 旧安装将自动备份
   - 将克隆全新的技能仓库
   - 如果你有 GitHub CLI，你将被提供 fork 的选项

4. **迁移个人技能**（如果有的话）：
   - 在你的本地技能仓库中创建分支
   - 从备份复制你的个人技能
   - 提交并推送到你的 fork
   - 考虑通过 PR 贡献回来

## 下一步

### 对于用户

- 探索新的问题解决技能
- 尝试基于分支的工作流进行技能改进
- 向社区贡献技能

### 对于贡献者

- 技能仓库现在位于 https://github.com/obra/superpowers-skills
- Fork → Branch → PR 工作流
- 参见 skills/meta/writing-skills/SKILL.md 了解文档的 TDD 方法

## 已知问题

目前没有。

## 致谢

- 受 Amplifier 模式启发的问题解决技能
- 社区贡献和反馈
- 对技能有效性的广泛测试和迭代

---

**完整变更日志：** https://github.com/obra/superpowers/compare/dd013f6...main
**技能仓库：** https://github.com/obra/superpowers-skills
**问题反馈：** https://github.com/obra/superpowers/issues
