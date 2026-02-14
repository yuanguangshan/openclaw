# agents/cache-trace.ts 解读

**文件路径**: `src/agents/cache-trace.ts`

## 文件用途
AI 代理执行过程缓存追踪模块，负责记录代理执行各个阶段的状态和数据，用于调试、性能分析和缓存优化。

## 主要类型定义

### CacheTraceStage
缓存追踪阶段类型
- `session:loaded` - 会话加载完成
- `session:sanitized` - 会话清理完成
- `session:limited` - 会话限制应用完成
- `prompt:before` - 提示生成前
- `prompt:images` - 图片处理阶段
- `stream:context` - 流式上下文阶段
- `session:after` - 会话处理后

### CacheTraceEvent
缓存追踪事件
- `ts: string` - 时间戳（ISO 8601）
- `seq: number` - 序列号
- `stage: CacheTraceStage` - 阶段标识
- `runId?: string` - 运行 ID
- `sessionId?: string` - 会话 ID
- `sessionKey?: string` - 会话键
- `provider?: string` - 提供商
- `modelId?: string` - 模型 ID
- `modelApi?: string | null` - 模型 API
- `workspaceDir?: string` - 工作区目录
- `prompt?: string` - 提示文本
- `system?: unknown` - 系统提示
- `options?: Record<string, unknown>` - 选项
- `model?: Record<string, unknown>` - 模型配置
- `messages?: AgentMessage[]` - 消息数组
- `messageCount?: number` - 消息数量
- `messageRoles?: Array<string | undefined>` - 消息角色
- `messageFingerprints?: string[]` - 消息指纹
- `messagesDigest?: string` - 消息摘要
- `systemDigest?: string` - 系统提示摘要
- `note?: string` - 备注
- `error?: string` - 错误信息

### CacheTrace
缓存追踪实例
- `enabled: true` - 启用状态
- `filePath: string` - 日志文件路径
- `recordStage: (stage, payload?) => void` - 记录阶段函数
- `wrapStreamFn: (streamFn) => StreamFn` - 包装流函数

## 主要函数

### createCacheTrace(params: CacheTraceInit): CacheTrace | null
- **功能**: 创建缓存追踪实例
- **参数**: 初始化参数（配置、环境、运行 ID 等）
- **返回值**: 缓存追踪实例或 null（未启用）
- **实现逻辑**:
  1. 解析缓存追踪配置
  2. 如果未启用，返回 null
  3. 获取或创建文件写入器
  4. 初始化序列号和基础事件数据
  5. 创建 `recordStage` 函数用于记录事件
  6. 创建 `wrapStreamFn` 函数用于包装流式调用
  7. 返回缓存追踪实例

### recordStage(stage: CacheTraceStage, payload?): void
- **功能**: 记录执行阶段事件
- **参数**:
  - `stage` - 阶段标识
  - `payload` - 附加数据
- **实现逻辑**:
  1. 创建事件对象（包含时间戳、序列号、阶段）
  2. 合并基础数据和负载数据
  3. 根据配置决定是否包含完整数据
  4. 对消息进行摘要处理
  5. 将事件序列化为 JSON 行
  6. 异步写入日志文件

### wrapStreamFn(streamFn): StreamFn
- **功能**: 包装流式函数以自动记录上下文
- **参数**: 原始流函数
- **返回值**: 包装后的流函数
- **实现逻辑**:
  1. 创建包装函数
  2. 在调用原始函数前记录 `stream:context` 事件
  3. 传递所有参数给原始函数
  4. 返回原始函数的结果

### stableStringify(value: unknown): string
- **功能**: 将值稳定地序列化为字符串（用于指纹生成）
- **参数**: 任意值
- **返回值**: 稳定的字符串表示
- **实现逻辑**:
  1. 处理基本类型（null、undefined、数字、字符串、布尔）
  2. 处理特殊类型（Error、Uint8Array、bigint）
  3. 处理数组（递归处理每个元素）
  4. 处理对象（键排序后序列化）
  5. 返回稳定的 JSON 字符串

### digest(value: unknown): string
- **功能**: 生成值的 SHA-256 摘要
- **参数**: 任意值
- **返回值**: 十六进制摘要字符串
- **实现逻辑**:
  1. 使用 `stableStringify` 序列化值
  2. 计算 SHA-256 哈希
  3. 返回十六进制字符串

### summarizeMessages(messages): ...
- **功能**: 生成消息摘要信息
- **参数**: 消息数组
- **返回值**: 摘要对象（数量、角色、指纹、整体摘要）
- **实现逻辑**:
  1. 为每条消息生成指纹
  2. 提取消息角色
  3. 计算整体消息摘要
  4. 返回摘要对象

## 主要依赖
- `node:crypto` - SHA-256 摘要计算
- `node:fs/promises` - 文件系统操作
- `node:path` - 路径处理
- `../config/config.js` - OpenClaw 配置
- `../config/paths.js` - 路径解析
- `../utils.js` - 工具函数
- `../utils/boolean.js` - 布尔值解析
- `../utils/safe-json.js` - 安全 JSON 序列化

## 使用场景
- 调试 AI 代理执行流程
- 性能分析和优化
- 缓存效果验证
- 问题诊断和排查
- 执行过程审计

## 代码行数
274 行

## 重要特性
- 异步日志写入（不阻塞主流程）
- 文件写入器缓存（避免重复创建）
- 稳定序列化（用于指纹生成）
- 消息摘要（避免记录完整消息）
- 可配置的数据包含（prompt、system、messages）
- 流式函数自动包装
- 环境变量和配置支持

## 配置选项

### 环境变量
- `OPENCLAW_CACHE_TRACE` - 启用/禁用缓存追踪
- `OPENCLAW_CACHE_TRACE_FILE` - 自定义日志文件路径
- `OPENCLAW_CACHE_TRACE_MESSAGES` - 是否包含完整消息
- `OPENCLAW_CACHE_TRACE_PROMPT` - 是否包含提示文本
- `OPENCLAW_CACHE_TRACE_SYSTEM` - 是否包含系统提示

### 配置文件
```typescript
{
  diagnostics: {
    cacheTrace: {
      enabled: true,
      filePath: "/path/to/cache-trace.jsonl",
      includeMessages: true,
      includePrompt: true,
      includeSystem: true,
    }
  }
}
```

## 默认行为
- **日志文件**: `<stateDir>/logs/cache-trace.jsonl`
- **包含消息**: 是
- **包含提示**: 是
- **包含系统提示**: 是

## 事件流程

### 典型执行流程
```
session:loaded → session:sanitized → session:limited
  ↓
prompt:before → prompt:images
  ↓
stream:context
  ↓
session:after
```

### 每个阶段的数据
- `session:loaded`: 加载的原始会话数据
- `session:sanitized`: 清理后的会话数据
- `session:limited`: 应用限制后的会话数据
- `prompt:before`: 生成的提示和选项
- `prompt:images`: 图片处理相关信息
- `stream:context`: 流式请求的上下文
- `session:after`: 更新后的会话数据

## 使用示例

### 创建缓存追踪
```typescript
const cacheTrace = createCacheTrace({
  cfg: config,
  runId: "run-123",
  sessionId: "session-456",
  sessionKey: "agent:default",
  provider: "anthropic",
  modelId: "claude-opus-4-6",
  modelApi: "messages",
  workspaceDir: "/workspace",
});

if (cacheTrace) {
  // 记录阶段
  cacheTrace.recordStage("session:loaded", {
    messages: messages,
    note: "Session loaded successfully",
  });

  // 包装流函数
  const wrappedStream = cacheTrace.wrapStreamFn(streamFn);
  const result = await wrappedStream(model, context, options);

  cacheTrace.recordStage("session:after", {
    note: "Session updated",
  });
}
```

### 日志输出示例
```json
{"ts":"2025-02-15T10:30:00.000Z","seq":1,"stage":"session:loaded","runId":"run-123","sessionId":"session-456","sessionKey":"agent:default","provider":"anthropic","modelId":"claude-opus-4-6","messageCount":5,"messageRoles":["user","assistant","user","assistant","user"],"messageFingerprints":["abc123...","def456...","ghi789...","jkl012...","mno345..."],"messagesDigest":"sha256hash..."}
{"ts":"2025-02-15T10:30:00.100Z","seq":2,"stage":"prompt:before","runId":"run-123","sessionId":"session-456","sessionKey":"agent:default","provider":"anthropic","modelId":"claude-opus-4-6","prompt":"You are a helpful assistant...","options":{"temperature":0.7,"maxTokens":4096}}
{"ts":"2025-02-15T10:30:00.200Z","seq":3,"stage":"stream:context","runId":"run-123","sessionId":"session-456","sessionKey":"agent:default","provider":"anthropic","modelId":"claude-opus-4-6","model":{"id":"claude-opus-4-6","provider":"anthropic","api":"messages"},"system":"You are a helpful assistant...","messageCount":6}
```

## 数据优化

### 摘要生成
- **消息指纹**: 每条消息的 SHA-256 摘要
- **整体摘要**: 所有消息指纹连接后的 SHA-256 摘要
- **系统提示摘要**: 系统提示的 SHA-256 摘要

### 摘要用途
- 检测数据变化（避免重复处理）
- 缓存键生成
- 数据一致性验证
- 调试问题追踪

## 文件写入器

### 特性
- 异步写入（不阻塞）
- 队列处理（保证顺序）
- 错误容错（失败不影响主流程）
- 单例模式（同一文件共享写入器）

### 实现细节
```typescript
// 获取或创建写入器
const writer = getWriter(filePath);

// 写入（异步）
writer.write(JSON.stringify(event) + "\n");
```

## 稳定序列化

### 目的
- 生成一致的字符串表示
- 用于指纹计算
- 避免因键顺序不同导致的指纹差异

### 实现特性
- 对象键排序
- 特殊类型处理（Error、Uint8Array、bigint）
- 数组递归处理
- 数字和布尔值标准化

## 性能考虑
- 异步写入不阻塞主流程
- 文件写入器缓存减少开销
- 消息摘要减少数据量
- 可配置的数据包含控制输出大小

## 安全考虑
- 文件路径解析使用 `resolveUserPath` 防止路径遍历
- 敏感数据通过配置控制是否记录
- 错误处理确保失败不影响主流程
