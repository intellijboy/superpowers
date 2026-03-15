# Claude Code Skills 测试

使用 Claude Code CLI 对 superpowers skills 进行自动化测试。

## 概述

此测试套件验证 skill 是否正确加载以及 Claude 是否按预期遵循它们。测试在无头模式（`claude -p`）下调用 Claude Code 并验证行为。

## 要求

- 已安装 Claude Code CLI 并在 PATH 中（`claude --version` 应该可以工作）
- 已安装本地 superpowers 插件（参见主 README 进行安装）

## 运行测试

### 运行所有快速测试（推荐）：
```bash
./run-skill-tests.sh
```

### 运行集成测试（慢，10-30 分钟）：
```bash
./run-skill-tests.sh --integration
```

### 运行特定测试：
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### 运行并显示详细输出：
```bash
./run-skill-tests.sh --verbose
```

### 设置自定义超时：
```bash
./run-skill-tests.sh --timeout 1800  # 集成测试 30 分钟
```

## 测试结构

### test-helpers.sh
Skill 测试的通用函数：
- `run_claude "prompt" [timeout]` - 使用提示运行 Claude
- `assert_contains output pattern name` - 验证模式存在
- `assert_not_contains output pattern name` - 验证模式不存在
- `assert_count output pattern count name` - 验证精确计数
- `assert_order output pattern_a pattern_b name` - 验证顺序
- `create_test_project` - 创建临时测试目录
- `create_test_plan project_dir` - 创建示例计划文件

### 测试文件

每个测试文件：
1. 引入 `test-helpers.sh`
2. 使用特定提示运行 Claude Code
3. 使用断言验证预期行为
4. 成功返回 0，失败返回非零

## 示例测试

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== 测试：我的 Skill ==="

# 询问 Claude 关于 skill
output=$(run_claude "my-skill skill 做什么？" 30)

# 验证响应
assert_contains "$output" "预期行为" "Skill 描述行为"

echo "=== 所有测试通过 ==="
```

## 当前测试

### 快速测试（默认运行）

#### test-subagent-driven-development.sh
测试 skill 内容和要求（约 2 分钟）：
- Skill 加载和可访问性
- 工作流顺序（spec 合规在代码质量之前）
- 自我评审要求已记录
- 计划阅读效率已记录
- Spec 合规评审器怀疑态度已记录
- 评审循环已记录
- 任务上下文提供已记录

### 集成测试（使用 --integration 标志）

#### test-subagent-driven-development-integration.sh
完整工作流执行测试（约 10-30 分钟）：
- 使用 Node.js 设置创建真实测试项目
- 创建包含 2 个任务的实现计划
- 使用 subagent-driven-development 执行计划
- 验证实际行为：
  - 计划在开始时读取一次（而非每个任务）
  - 完整任务文本在 subagent 提示中提供
  - Subagent 在报告前执行自我评审
  - Spec 合规评审在代码质量之前发生
  - Spec 评审器独立阅读代码
  - 产生可工作的实现
  - 测试通过
  - 创建正确的 git 提交

**它测试什么：**
- 工作流实际上端到端工作
- 我们的改进实际被应用
- Subagent 正确遵循 skill
- 最终代码功能正常且经过测试

## 添加新测试

1. 创建新测试文件：`test-<skill-name>.sh`
2. 引入 test-helpers.sh
3. 使用 `run_claude` 和断言编写测试
4. 添加到 `run-skill-tests.sh` 中的测试列表
5. 使其可执行：`chmod +x test-<skill-name>.sh`

## 超时考虑

- 默认超时：每个测试 5 分钟
- Claude Code 可能需要时间响应
- 如需要用 `--timeout` 调整
- 测试应该聚焦以避免长时间运行

## 调试失败的测试

使用 `--verbose`，你将看到完整的 Claude 输出：
```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

不使用 verbose，只有失败显示输出。

## CI/CD 集成

在 CI 中运行：
```bash
# 为 CI 环境使用显式超时运行
./run-skill-tests.sh --timeout 900

# 退出码 0 = 成功，非零 = 失败
```

## 注意事项

- 测试验证 skill *指令*，而非完整执行
- 完整工作流测试会很慢
- 聚焦于验证关键 skill 要求
- 测试应该是确定性的
- 避免测试实现细节
