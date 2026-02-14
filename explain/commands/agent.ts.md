# commands/agent.ts 解读

**文件路径**: `src/commands/agent.ts`

## 文件用途
代理命令主入口，处理通过 OpenClaw 代理执行 AI 任务的所有操作。

## 主要函数

### agentCommand(opts, deps, runtime)
- **功能**: 执行代理命令
- **参数**:
  - `opts`: 命令选项
    - `message`: 消息内容（必需）
    - `to`: 目标用户/会话
    - `sessionId`: 会话 ID
    - `sessionKey`: 会话键
    - `agentId`: 代理 ID
    - `thinking`: 思考级别（off/low/medium/high/xhigh）
    - `thinkingOnce`: 一次性思考级别
    - `verbose`: 详细级别（on/full/off）
    - `timeout`: 超时时间（秒）
    - `deliver`: 是否投递结果
    - `images`: 图像输入
    - `streamParams`: 流式参数
    - `clientTools`: 客户端工具
    - `extraSystemPrompt`: 额外系统提示
    - `inputPrecedence`: 输入优先级
    - `lane`: 车道选择
  - `deps`: CLI 依赖
  - `runtime`: 运行时环境

## 主要处理流程

### 1. 参数验证
- 检查必需的消息内容
- 验证至少有一个目标标识（to/sessionId/sessionKey/agentId）
- 验证代理 ID 有效性
- 验证会话键和代理 ID 匹配

### 2. 配置解析
- 加载全局配置
- 解析代理 ID（覆盖或会话键）
- 解析工作空间目录
- 确保工作空间
- 解析配置的模型
- 解析思考级别提示

### 3. 思考和详细级别处理
- 验证思考级别有效性
- 验证一次性思考级别有效性
- 验证详细级别有效性

### 4. 超时处理
- 解析超时秒数
- 验证超时值有效性
- 计算毫秒超时

### 5. 会话解析
- 解析会话标识符
- 获取会话条目和存储
- 检查投递策略

### 6. 技能快照处理
- 判断是否需要技能快照
- 构建工作空间技能快照
- 更新会话存储中的技能快照

### 7. 模型配置处理
- 解析代理主模型
- 处理代理级模型配置
- 加载模型目录
- 构建允许的模型集
- 处理会话存储的模型覆盖
- 处理认证配置文件覆盖

### 8. 思考级别解析
- 解析思考级别（命令参数 → 会话存储 → 默认配置）
- 验证 xhigh 思考级别的支持
- 更新会话存储

### 9. 认证配置文件处理
- 验证认证配置文件匹配当前提供商
- 清理无效的认证配置文件覆盖

### 10. 代理执行
- 根据提供商选择执行模式：
  - CLI 提供商：调用 `runCliAgent`
  - 嵌入式提供商：调用 `runEmbeddedPiAgent`
- 处理模型回退
- 跟踪生命周期事件

### 11. 结果投递
- 更新会话存储（令牌、模型、回退信息）
- 构建投递载荷
- 调用投递函数

## 主要依赖
- `./agent/types.js` - 代理命令类型
- `../agents/agent-scope.js` - 代理作用域
- `../agents/auth-profiles.js` - 认证配置文件
- `../agents/cli-runner.js` - CLI 运行器
- `../agents/cli-session.js` - CLI 会话
- `../agents/defaults.js` - 默认值
- `../agents/model-catalog.js` - 模型目录
- `../agents/model-fallback.js` - 模型回退
- `../agents/model-selection.js` - 模型选择
- `../agents/pi-embedded.js` - 嵌入式 AI
- `../agents/skills.js` - 技能
- `../agents/skills/refresh.js` - 技能刷新
- `../agents/timeout.js` - 超时
- `../agents/workspace.js` - 工作空间
- `../auto-reply/thinking.js` - 思考级别
- `../cli/command-format.js` - CLI 格式化
- `../cli/deps.js` - CLI 依赖
- `../config/config.js` - 配置管理
- `../config/sessions.js` - 会话配置
- `../infra/agent-events.js` - 代理事件
- `../infra/skills-remote.js` - 远程技能
- `../routing/session-key.js` - 会话键
- `../runtime.js` - 运行时
- `../sessions/level-overrides.js` - 级别覆盖
- `../sessions/model-overrides.js` - 模型覆盖
- `../sessions/send-policy.js` - 投递策略
- `../utils/message-channel.js` - 消息通道
- `./agent/delivery.js` - 代理投递
- `./agent/run-context.js` - 运行上下文
- `./agent/session-store.js` - 会话存储
- `./agent/session.js` - 会话解析

## 使用场景
- 通过 CLI 发送消息给 AI
- 代理任务执行
- 模型回退测试
- 技能快照管理
- 会话状态跟踪

## 代码行数
530 行

## 重要特性
- 多提供商支持
- 模型回退机制
- 技能快照缓存
- 思考级别控制
- 详细级别调整
- 超时控制
- 会话状态持久化

## 思考级别 (ThinkLevel)
- `off` - 无思考
- `low` - 低思考
- `medium` - 中等思考
- `high` - 高思考
- `xhigh` - 极高思考（仅特定模型支持）

## 详细级别 (VerboseLevel)
- `on` - 开启
- `full` - 完全
- `off` - 关闭

## 使用示例
```bash
# 基本使用
openclaw agent --message "Hello, how are you?"

# 指定目标用户
openclaw agent --message "What's the weather?" --to user@example.com

# 使用会话 ID
openclaw agent --message "Continue our conversation" --session-id abc123

# 设置思考级别
openclaw agent --message "Explain this step by step" --thinking high

# 一次性思考
openclaw agent --message "Quick answer" --thinking-once low

# 设置详细输出
openclaw agent --message "Show me everything" --verbose full

# 设置超时
openclaw agent --message "Quick question" --timeout 30

# 使用特定代理
openclaw agent --message "Help me with coding" --agent-id coder

# 添加图像
openclaw agent --message "Describe this image" --images ./image.jpg

# 流式参数
openclaw agent --message "Stream response" --stream-params '{"max_tokens": 100}'

# 不投递（仅执行）
openclaw agent --message "Process this" --no-deliver

# 工作空间路径
openclaw agent --message "Use my project" --workspace /path/to/project
```

## 错误处理
- 缺少消息内容 → 抛出错误
- 缺少目标标识 → 抛出错误
- 无效的代理 ID → 抛出错误
- 会话键和代理 ID 不匹配 → 抛出错误
- 无效的思考级别 → 抛出错误
- 无效的详细级别 → 抛出错误
- 无效的超时值 → 抛出错误
- 投递被拒绝 → 抛出错误

## 模型回退逻辑
1. 尝试主模型
2. 如果失败，尝试回退模型
3. 支持多个回退模型
4. 记录回退提供商和模型
5. 更新会话存储中的回退信息