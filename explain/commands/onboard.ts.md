# commands/onboard.ts 解读

**文件路径**: `src/commands/onboard.ts`

## 文件用途
入驻（onboarding）命令主入口，处理首次使用 OpenClaw 的设置流程。

## 主要函数

### onboardCommand(opts, runtime)
- **功能**: 执行入驻流程
- **参数**:
  - `opts`: 入驻选项
    - `authChoice`: 认证方式
    - `flow`: 流程类型（manual/advanced）
    - `nonInteractive`: 非交互模式
    - `acceptRisk`: 接受风险
    - `reset`: 重置
    - `workspace`: 工作空间路径
  - `runtime`: 运行时环境
- **实现逻辑**:
  1. 检查运行时支持
  2. 规范化认证方式选择
  3. 检查弃用的认证方式
  4. 验证非交互模式的风险接受
  5. 处理重置操作
  6. Windows 平台提示（推荐 WSL2）
  7. 根据模式选择流程（交互式/非交互式）

## 主要依赖
- `../runtime.js` - 运行时类型和默认值
- `../cli/command-format.js` - CLI 命令格式化
- `../config/config.js` - 配置文件操作
- `../infra/runtime-guard.js` - 运行时支持检查
- `../utils.js` - 工具函数
- `./auth-choice-legacy.js` - 旧版认证选择
- `./onboard-types.js` - 入驻类型定义
- `./onboard-helpers.js` - 入驻帮助函数
- `./onboard-interactive.js` - 交互式入驻
- `./onboard-non-interactive.js` - 非交互式入驻

## 使用场景
- 首次使用 OpenClaw
- 系统初始化设置
- 批量自动化配置
- 非交互式安装

## 代码行数
79 行

## 重要特性
- 交互式和非交互式双模式
- 多种认证方式支持
- 风险警告和确认
- 平台特定提示
- 重置和恢复功能

## 入驻流程类型

### 认证方式 (authChoice)
- `token` - Token 认证（推荐）
- `openai-codex` - OpenAI Codex OAuth
- `claude-cli` - Claude CLI（已弃用）
- `codex-cli` - Codex CLI（已弃用）

### 流程类型 (flow)
- `manual` - 手动/高级流程
- `advanced` - 高级配置流程
- 未指定 - 使用默认流程

### 弃用的认证方式
以下认证方式已弃用，建议迁移：
- `claude-cli` → 使用 `token`（setup-token）
- `codex-cli` → 使用 `openai-codex`

## 交互式流程
1. 检查运行时环境
2. 显示欢迎信息
3. 选择认证方式
4. 配置模型和提供商
5. 配置网关
6. 配置通道
7. 配置代理
8. 测试连接
9. 完成入驻

## 非交互式流程
1. 使用命令行参数提供所有配置
2. 验证必需参数
3. 创建配置文件
4. 验证配置
5. 完成

## 使用示例
```bash
# 交互式入驻（默认）
openclaw onboard

# 使用 Token 认证
openclaw onboard --auth-choice token

# 非交互式模式（需要显式接受风险）
openclaw onboard --non-interactive --accept-risk --flow advanced

# 使用特定认证方式
openclaw onboard --auth-choice openai-codex

# 重置配置
openclaw onboard --reset --workspace /path/to/workspace
```

## 风险警告
非交互式模式需要显式接受风险，因为：
- 可能跳过重要的验证步骤
- 可能创建不安全的配置
- 可能导致数据丢失

## 平台注意事项
- Windows：推荐使用 WSL2 获得更好的体验
- 原生 Windows 支持可能有限制
- 提供了相关文档链接

## 错误处理
- 运行时不支持 → 退出并显示错误
- 非交互模式未接受风险 → 退出并显示错误
- 配置验证失败 → 显示详细错误信息

## 相关文档
- 安全文档: https://docs.openclaw.ai/security
- Windows 指南: https://docs.openclaw.ai/windows
- 配置文档: https://docs.openclaw.ai/configuration