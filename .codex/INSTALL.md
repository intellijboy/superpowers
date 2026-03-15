# 为 Codex 安装 Superpowers

通过原生 skill 发现机制在 Codex 中启用 superpowers 技能。只需克隆和符号链接。

## 前置条件

- Git

## 安装

1. **克隆 superpowers 仓库：**
   ```bash
   git clone https://github.com/obra/superpowers.git ~/.codex/superpowers
   ```

2. **创建 skills 符号链接：**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
   ```

   **Windows (PowerShell)：**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\superpowers\skills"
   ```

3. **重启 Codex**（退出并重新启动 CLI）以发现技能。

## 从旧版引导迁移

如果你在原生 skill 发现功能之前安装了 superpowers，你需要：

1. **更新仓库：**
   ```bash
   cd ~/.codex/superpowers && git pull
   ```

2. **创建 skills 符号链接**（上面的步骤 2）— 这是新的发现机制。

3. **从 `~/.codex/AGENTS.md` 中移除旧的引导块** — 任何引用 `superpowers-codex bootstrap` 的块都不再需要。

4. **重启 Codex。**

## 验证

```bash
ls -la ~/.agents/skills/superpowers
```

你应该看到一个指向你的 superpowers skills 目录的符号链接（或在 Windows 上的连接点）。

## 更新

```bash
cd ~/.codex/superpowers && git pull
```

技能通过符号链接即时更新。

## 卸载

```bash
rm ~/.agents/skills/superpowers
```

可选择删除克隆：`rm -rf ~/.codex/superpowers`。
