# commands/message.ts 解读

**文件路径**: `src/commands/message.ts`

## 文件用途
消息命令模块，处理通过网关发送消息的各种操作。

## 主要函数

### messageCommand(opts, deps, runtime)
- **功能**: 执行消息命令
- **参数**:
  - `opts`: 命令选项
  - `deps`: CLI 依赖
  - `runtime`: 运行时环境
- **支持的动作**:
  - `send` - 发送消息
  - `poll` - 轮询消息
- **实现逻辑**:
  1. 加载配置
  2. 解析动作参数（默认 send）
  3. 验证动作有效性
  4. 创建外发依赖
  5. 运行消息动作
  6. 处理输出格式（JSON/文本）
  7. 显示进度（如果需要）

## 主要依赖
- `../infra/outbound/deliver.js` - 外发投递类型
- `../runtime.js` - 运行时类型
- `../channels/plugins/types.js` - 通道消息动作类型
- `../cli/outbound-send-deps.js` - 外发依赖创建
- `../cli/progress.js` - 进度显示
- `../config/config.js` - 配置加载
- `../infra/outbound/message-action-runner.js` - 消息动作运行器
- `../utils/message-channel.js` - 消息通道工具
- `./message-format.js` - 消息格式化

## 使用场景
- 通过 CLI 发送消息
- 轮询消息
- 网关消息操作
- 批量消息处理

## 代码行数
68 行

## 重要特性
- 支持多种消息动作
- JSON 和文本输出格式
- 进度指示器支持
- 干运行模式支持

## 使用示例
```bash
# 发送消息
openclaw message send --to user@example.com --message "Hello"

# 轮询消息
openclaw message poll --channel telegram

# JSON 输出
openclaw message send --to user@example.com --json

# 干运行
openclaw message send --to user@example.com --dry-run
```