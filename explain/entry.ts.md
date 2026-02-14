# entry.ts 解读

**文件路径**: `src/entry.ts`

## 文件用途
这是 OpenClaw CLI 的主入口点文件。它负责初始化命令行界面、处理环境变量配置、管理进程重生机制等核心功能。

## 主要功能

### 1. 进程初始化
- 设置进程标题为 "openclaw"
- 安装进程警告过滤器
- 规范化环境变量

### 2. 实验性警告抑制
通过 `ensureExperimentalWarningSuppressed()` 函数管理 Node.js 实验性警告的显示，避免在命令行中显示不必要的警告信息。

### 3. 进程重生机制
当需要抑制实验性警告时，会通过 `spawn()` 创建新的子进程，并将当前进程的控制权转移给子进程。

### 4. CLI 配置文件处理
- 解析 CLI profile 参数 (`parseCliProfileArgs`)
- 应用 CLI profile 环境变量 (`applyCliProfileEnv`)
- 规范化 Windows 参数 (`normalizeWindowsArgv`)

### 5. 主程序启动
- 导入并执行 `./cli/run-main.js` 中的 `runCli` 函数
- 处理启动过程中的错误

## 主要依赖
- `node:child_process` - 进程创建和管理
- `node:process` - 进程相关 API
- `./cli/profile.js` - CLI profile 管理
- `./cli/respawn-policy.js` - 重生策略
- `./cli/windows-argv.js` - Windows 参数处理
- `./infra/env.js` - 环境变量处理
- `./infra/warning-filter.js` - 警告过滤器
- `./process/child-process-bridge.js` - 子进程桥接

## 重要逻辑
1. **环境变量标准化**: 确保所有环境变量都使用一致的格式
2. **实验性警告管理**: 通过重生进程来添加 `--disable-warning=ExperimentalWarning` 标志
3. **错误处理**: 在进程重生和 CLI 启动过程中都有完善的错误处理
4. **Windows 兼容性**: 专门处理 Windows 平台的参数规范化

## 代码行数
109 行

## 关键设计决策
- 使用进程重生而不是直接修改 NODE_OPTIONS，因为某些标志不允许在 NODE_OPTIONS 中使用
- 环境变量 `OPENCLAW_NODE_OPTIONS_READY` 用于防止无限递归重生
- 继承子进程的标准 I/O，确保输出一致