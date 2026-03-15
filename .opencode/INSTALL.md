# 为 OpenCode 安装 Superpowers

## 前置条件

- 已安装 [OpenCode.ai](https://opencode.ai)
- 已安装 Git

## 安装步骤

### 1. 克隆 Superpowers

```bash
git clone https://github.com/obra/superpowers.git ~/.config/opencode/superpowers
```

### 2. 注册插件

创建符号链接以便 OpenCode 发现插件：

```bash
mkdir -p ~/.config/opencode/plugins
rm -f ~/.config/opencode/plugins/superpowers.js
ln -s ~/.config/opencode/superpowers/.opencode/plugins/superpowers.js ~/.config/opencode/plugins/superpowers.js
```

### 3. 符号链接技能

创建符号链接以便 OpenCode 的原生 skill 工具发现 superpowers 技能：

```bash
mkdir -p ~/.config/opencode/skills
rm -rf ~/.config/opencode/skills/superpowers
ln -s ~/.config/opencode/superpowers/skills ~/.config/opencode/skills/superpowers
```

### 4. 重启 OpenCode

重启 OpenCode。插件将自动注入 superpowers 上下文。

通过询问验证："你有 superpowers 吗？"

## 使用方法

### 查找技能

使用 OpenCode 的原生 `skill` 工具列出可用技能：

```
使用 skill 工具列出技能
```

### 加载技能

使用 OpenCode 的原生 `skill` 工具加载特定技能：

```
使用 skill 工具加载 superpowers/brainstorming
```

### 个人技能

在 `~/.config/opencode/skills/` 中创建你自己的技能：

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

创建 `~/.config/opencode/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: 当 [条件] 时使用 - [功能描述]
---

# 我的技能

[你的技能内容在这里]
```

### 项目技能

在项目内的 `.opencode/skills/` 中创建项目特定的技能。

**技能优先级：** 项目技能 > 个人技能 > Superpowers 技能

## 更新

```bash
cd ~/.config/opencode/superpowers
git pull
```

## 故障排除

### 插件未加载

1. 检查插件符号链接：`ls -l ~/.config/opencode/plugins/superpowers.js`
2. 检查源文件是否存在：`ls ~/.config/opencode/superpowers/.opencode/plugins/superpowers.js`
3. 检查 OpenCode 日志中的错误

### 技能未找到

1. 检查技能符号链接：`ls -l ~/.config/opencode/skills/superpowers`
2. 验证它指向：`~/.config/opencode/superpowers/skills`
3. 使用 `skill` 工具列出已发现的内容

### 工具映射

当技能引用 Claude Code 工具时：
- `TodoWrite` → `todowrite`
- 带有 subagent 的 `Task` → `@mention` 语法
- `Skill` 工具 → OpenCode 的原生 `skill` 工具
- 文件操作 → 你的原生工具

## 获取帮助

- 问题反馈：https://github.com/obra/superpowers/issues
- 完整文档：https://github.com/obra/superpowers/blob/main/docs/README.opencode.md
