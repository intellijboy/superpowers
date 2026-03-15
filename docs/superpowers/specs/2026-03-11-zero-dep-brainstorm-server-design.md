# 零依赖 Brainstorm 服务器

用单个零依赖的 `server.js` 替换 brainstorm companion 服务器的 vendored node_modules（express、ws、chokidar — 714 个跟踪文件），仅使用 Node.js 内置模块。

## 动机

将 node_modules vendor 到 git 仓库中会创建供应链风险：冻结的依赖项无法获得安全补丁，714 个第三方代码文件在未经审计的情况下被提交，对 vendored 代码的修改看起来像正常的提交。虽然实际风险很低（仅 localhost 开发服务器），但消除它是直接的。

## 架构

单个 `server.js` 文件（约 250-300 行），使用 `http`、`crypto`、`fs` 和 `path`。该文件服务两个角色：

- **直接运行时**（`node server.js`）：启动 HTTP/WebSocket 服务器
- **被引用时**（`require('./server.js')`）：导出 WebSocket 协议函数供单元测试使用

### WebSocket 协议

仅实现 RFC 6455 的文本帧：

**握手：** 使用 SHA-1 + RFC 6455 魔术 GUID 从客户端的 `Sec-WebSocket-Key` 计算 `Sec-WebSocket-Accept`。返回 101 Switching Protocols。

**帧解码（客户端到服务器）：** 处理三种掩码长度编码：
- 小：payload < 126 字节
- 中：126-65535 字节（16 位扩展）
- 大：> 65535 字节（64 位扩展）

使用 4 字节掩码密钥进行 XOR 解掩码 payload。返回 `{ opcode, payload, bytesConsumed }` 或 `null` 表示不完整的缓冲区。拒绝未掩码的帧。

**帧编码（服务器到客户端）：** 未掩码的帧，使用相同的三种长度编码。

**处理的 Opcodes：** TEXT (0x01)、CLOSE (0x08)、PING (0x09)、PONG (0x0A)。未识别的 opcode 获得状态码 1003（不支持的数据）的关闭帧。

**故意跳过：** 二进制帧、分片消息、扩展（permessage-deflate）、子协议。这些对于 localhost 客户端之间的小型 JSON 文本消息是不必要的。扩展和子协议在握手中协商 — 通过不广播它们，它们永远不会激活。

**缓冲区累积：** 每个连接维护一个缓冲区。在 `data` 事件时，追加并循环调用 `decodeFrame` 直到它返回 null 或缓冲区为空。

### HTTP 服务器

三个路由：

1. **`GET /`** — 按 mtime 从 screen 目录提供最新的 `.html`。检测完整文档 vs 片段，将片段包装在 frame 模板中，注入 helper.js。返回 `text/html`。当没有 `.html` 文件存在时，提供一个硬编码的等待页面（"Waiting for Claude to push a screen..."），并注入 helper.js。
2. **`GET /files/*`** — 从 screen 目录提供静态文件，使用硬编码扩展名映射（html、css、js、png、jpg、gif、svg、json）进行 MIME 类型查找。如果未找到返回 404。
3. **其他所有** — 404。

WebSocket 升级通过 HTTP 服务器上的 `'upgrade'` 事件处理，与请求处理器分开。

### 配置

环境变量（全部可选）：

- `BRAINSTORM_PORT` — 绑定的端口（默认：随机高端口 49152-65535）
- `BRAINSTORM_HOST` — 绑定的接口（默认：`127.0.0.1`）
- `BRAINSTORM_URL_HOST` — 启动 JSON 中 URL 的主机名（默认：当 host 是 `127.0.0.1` 时为 `localhost`，否则与 host 相同）
- `BRAINSTORM_DIR` — screen 目录路径（默认：`/tmp/brainstorm`）

### 启动序列

1. 如果 `SCREEN_DIR` 不存在则创建它（`mkdirSync` recursive）
2. 从 `__dirname` 加载 frame 模板和 helper.js
3. 在配置的 host/port 上启动 HTTP 服务器
4. 在 `SCREEN_DIR` 上启动 `fs.watch`
5. 成功监听后，将 `server-started` JSON 记录到 stdout：`{ type, port, host, url_host, url, screen_dir }`
6. 将相同的 JSON 写入 `SCREEN_DIR/.server-info`，以便当 stdout 被隐藏（后台执行）时 agent 可以找到连接详情

### 应用层 WebSocket 消息

当从客户端收到 TEXT 帧时：

1. 解析为 JSON。如果解析失败，记录到 stderr 并继续。
2. 记录到 stdout 作为 `{ source: 'user-event', ...event }`。
3. 如果事件包含 `choice` 属性，将 JSON 追加到 `SCREEN_DIR/.events`（每个事件一行）。

### 文件监视

`fs.watch(SCREEN_DIR)` 替代 chokidar。在 HTML 文件事件时：

- 新文件时（文件存在的 `rename` 事件）：如果存在则删除 `.events` 文件（`unlinkSync`），将 `screen-added` 作为 JSON 记录到 stdout
- 文件更改时（`change` 事件）：将 `screen-updated` 作为 JSON 记录到 stdout（不清除 `.events`）
- 两种事件：向所有连接的 WebSocket 客户端发送 `{ type: 'reload' }`

按文件名使用约 100ms 超时进行防抖，以防止重复事件（在 macOS 和 Linux 上常见）。

### 错误处理

- 来自 WebSocket 客户端的格式错误 JSON：记录到 stderr，继续
- 未处理的 opcode：用状态码 1003 关闭
- 客户端断开连接：从广播集合中移除
- `fs.watch` 错误：记录到 stderr，继续
- 无优雅关闭逻辑 — shell 脚本通过 SIGTERM 处理进程生命周期

## 变化内容

| 之前 | 之后 |
|---|---|
| `index.js` + `package.json` + `package-lock.json` + 714 个 `node_modules` 文件 | `server.js`（单文件） |
| express、ws、chokidar 依赖 | 无 |
| 无静态文件服务 | `/files/*` 从 screen 目录提供 |

## 保持不变

- `helper.js` — 无变化
- `frame-template.html` — 无变化
- `start-server.sh` — 一行更新：`index.js` 改为 `server.js`
- `stop-server.sh` — 无变化
- `visual-companion.md` — 无变化
- 所有现有服务器行为和外部契约

## 平台兼容性

- `server.js` 仅使用跨平台的 Node 内置模块
- `fs.watch` 在 macOS、Linux 和 Windows 上的单层平目录中可靠
- Shell 脚本需要 bash（Windows 上的 Git Bash，这是 Claude Code 所需的）

## 测试

**单元测试**（`ws-protocol.test.js`）：通过引用 `server.js` 导出直接测试 WebSocket 帧编码/解码、握手计算和协议边界情况。

**集成测试**（`server.test.js`）：测试完整的服务器行为 — HTTP 服务、WebSocket 通信、文件监视、brainstorming 工作流。使用 `ws` npm 包作为仅测试的客户端依赖（不分发给最终用户）。
