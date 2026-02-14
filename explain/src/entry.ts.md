# 文件解释：src/entry.ts

## 文件路径
`src/entry.ts`

## 文件用途
OpenClaw CLI 的主入口点，负责 CLI 进程的初始化和环境设置。主要功能包括：
- 设置进程标题
- 安装警告过滤器
- 标准化环境变量
- 处理颜色输出配置
- 确保 Node.js 实验性警告被抑制
- 通过重启机制传递正确的 Node.js 选项

## 主要类/函数

### `hasExperimentalWarningSuppressed(): boolean`
检查实验性警告是否已被抑制，通过检查 `NODE_OPTIONS` 环境变量和 `process.execArgv`。

### `ensureExperimentalWarningSuppressed(): boolean`
确保实验性警告被抑制。如果需要，通过生成带有 `--disable-warning` 标志的子进程来重启 CLI，并传递所有参数。返回 `true` 表示已重启，`false` 表示可以继续执行。

## 主要依赖

- `node:child_process` - 用于生成子进程
- `node:process` - 进程管理
- `./cli/profile.js` - CLI 配置文件处理
- `./cli/respawn-policy.js` - 重启策略判断
- `./cli/windows-argv.js` - Windows 参数标准化
- `./infra/env.js` - 环境变量处理
- `./infra/warning-filter.js` - 警告过滤器
- `./process/child-process-bridge.js` - 子进程桥接

## 重要逻辑说明

1. **进程初始化**：设置进程标题为 "openclaw"，安装警告过滤器，标准化环境变量。

2. **颜色控制**：检测 `--no-color` 标志，设置相应的环境变量来禁用颜色输出。

3. **警告抑制机制**：
   - 检查是否已经抑制了实验性警告（通过环境变量或 execArgv）
   - 如果未抑制且需要抑制，生成新的子进程
   - 子进程使用 `--disable-warning=ExperimentalWarning` 标志
   - 通过 `OPENCLAW_NODE_OPTIONS_READY` 环境变量防止无限重启
   - 继承 stdin/stdout/stderr 以保持用户体验

4. **参数处理**：标准化 Windows 参数，解析 CLI 配置文件参数，应用配置文件环境变量。

5. **CLI 启动**：如果不需要重启，导入并运行 CLI 主模块。

## 行数统计
109 行
