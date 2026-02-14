# commands/reset.ts 解读

**文件路径**: `src/commands/reset.ts`

## 文件用途
重置命令模块，用于清理 OpenClaw 配置、凭据和会话数据。

## 主要类型定义

### ResetScope
重置范围：
- `"config"` - 仅配置
- `"config+creds+sessions"` - 配置、凭据和会话
- `"full"` - 完全重置（包括工作空间）

### ResetOptions
重置选项：
- `scope`: 重置范围
- `yes`: 自动确认（非交互模式）
- `nonInteractive`: 非交互模式
- `dryRun`: 干运行（不实际删除）

## 主要函数

### stopGatewayIfRunning(runtime)
- **功能**: 如果网关正在运行则停止
- **参数**: 运行时环境
- **实现逻辑**:
  1. 检查是否为 Nix 模式
  2. 检查网关服务是否加载
  3. 停止网关服务
  - 错误时显示错误信息

### resetCommand(runtime, opts)
- **功能**: 执行重置命令
- **参数**:
  - `runtime`: 运行时环境
  - `opts`: 重置选项
- **实现逻辑**:
  1. 验证非交互模式的必要参数
  2. 选择重置范围
  3. 验证重置范围有效性
  4. 确认重置操作
  5. 根据范围执行重置

### 重置范围处理

#### scope = "config"
- 停止网关（如果运行中）
- 删除配置文件
- 显示配置文件路径

#### scope = "config+creds+sessions"
- 停止网关（如果运行中）
- 删除配置文件
- 删除 OAuth 目录
- 删除所有代理会话目录
- 显示删除路径
- 提示重新运行入驻

#### scope = "full"
- 停止网关（如果运行中）
- 删除配置文件
- 删除 OAuth 目录
- 删除所有代理会话目录
- 删除所有工作空间目录
- 显示删除路径
- 提示重新运行入驻

## 主要依赖
- `../runtime.js` - 运行时类型
- `@clack/prompts` - 提示库
- `../cli/command-format.js` - CLI 命令格式化
- `../config/config.js` - 配置路径
- `../config/config.js` - 配置加载
- `../daemon/service.js` - 网关服务
- `../terminal/prompt-style.js` - 提示样式
- `./cleanup-utils.js` - 清理工具
  - `collectWorkspaceDirs` - 收集工作空间目录
  - `isPathWithin` - 路径检查
  - `listAgentSessionDirs` - 列出代理会话目录
  - `removePath` - 删除路径

## 使用场景
- 完全重置 OpenClaw 安装
- 清除配置问题
- 切换到全新配置
- 排除会话相关的问题

## 代码行数
169 行

## 重要特性
- 安全确认（防止意外删除）
- 非交互模式支持
- 干运行模式
- 分层重置范围
- 完善的错误处理
- 详细的删除日志

## 使用示例
```bash
# 仅重置配置
openclaw reset --scope config

# 重置配置、凭据和会话
openclaw reset --scope config+creds+sessions

# 完全重置
openclaw reset --scope full

# 非交互模式（自动确认）
openclaw reset --yes

# 干运行（显示将要删除的内容）
openclaw reset --dry-run

# 组合使用
openclaw reset --scope full --yes
```

## 危险提示
此命令会永久删除数据，使用前请确保：
1. 已备份重要配置
2. 了解将删除的内容
3. 在测试环境先尝试

## 清理的文件和目录

### 配置
- `~/.openclaw/config.json` 或 `config.yaml`

### 认证凭据
- `~/.openclaw/credentials/` 目录

### OAuth 配置
- `~/.openclaw/oauth/` 目录

### 会话数据
- `~/.openclaw/sessions/` 或代理特定的会话目录

### 工作空间
- 各代理配置的工作空间目录

## 错误处理
- 非交互模式缺少必要参数 → 退出并显示错误
- 无效的重置范围 → 退出并显示错误
- 用户取消重置 → 取消操作
- 删除失败 → 显示详细错误信息
- 网关停止失败 → 显示错误但继续操作

## 网关服务集成
- 在 Nix 系统上通过 systemd 管理
- 自动检测和停止运行的网关服务
- 防止删除时服务冲突