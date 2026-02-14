# gateway/hooks.ts 解读

**文件路径**: `src/gateway/hooks.ts`

## 文件用途
Webhook 配置和请求处理模块，负责解析和验证 Webhook 请求，处理认证、会话策略、代理策略和消息通道映射。

## 主要类型定义

### HooksConfigResolved
解析后的 Webhook 配置
- `basePath: string` - 基础路径（默认 `/hooks`）
- `token: string` - 认证令牌
- `maxBodyBytes: number` - 最大请求体大小
- `mappings: HookMappingResolved[]` - 映射规则
- `agentPolicy: HookAgentPolicyResolved` - 代理策略
- `sessionPolicy: HookSessionPolicyResolved` - 会话策略

### HookAgentPolicyResolved
代理策略配置
- `defaultAgentId: string` - 默认代理 ID
- `knownAgentIds: Set<string>` - 已知代理 ID 集合
- `allowedAgentIds?: Set<string>` - 允许的代理 ID 集合

### HookSessionPolicyResolved
会话策略配置
- `defaultSessionKey?: string` - 默认会话键
- `allowRequestSessionKey: boolean` - 是否允许请求指定会话键
- `allowedSessionKeyPrefixes?: string[]` - 允许的会话键前缀

### HookAgentPayload
Webhook 代理请求负载
- `message: string` - 消息内容
- `name: string` - Webhook 名称
- `agentId?: string` - 目标代理 ID
- `wakeMode: "now" | "next-heartbeat"` - 唤醒模式
- `sessionKey?: string` - 会话键
- `deliver: boolean` - 是否投递消息
- `channel: HookMessageChannel` - 消息通道
- `to?: string` - 目标地址
- `model?: string` - 模型
- `thinking?: string` - 思考级别
- `timeoutSeconds?: number` - 超时秒数

### HookMessageChannel
Webhook 消息通道类型（所有支持通道 + "last"）

## 主要函数

### resolveHooksConfig(cfg: OpenClawConfig): HooksConfigResolved | null
- **功能**: 解析和验证 Webhook 配置
- **参数**: OpenClaw 配置对象
- **返回值**: 解析后的配置对象或 null（未启用）
- **实现逻辑**:
  1. 检查 hooks 是否启用
  2. 验证 token 必须存在
  3. 标准化路径（添加前导斜杠，移除尾部斜杠）
  4. 验证路径不能为根路径 `/`
  5. 设置最大请求体大小（默认 256KB）
  6. 解析映射规则
  7. 解析代理策略（默认代理、已知代理、允许的代理）
  8. 解析会话策略（默认会话键、前缀规则）
  9. 验证配置一致性（默认会话键必须匹配前缀规则）
  10. 返回完整配置对象

### extractHookToken(req: IncomingMessage): string | undefined
- **功能**: 从 HTTP 请求中提取 Webhook 认证令牌
- **参数**: HTTP 请求对象
- **返回值**: 认证令牌或 undefined
- **实现逻辑**:
  1. 首先尝试从 Authorization header 提取 Bearer token
  2. 如果不存在，尝试从 `x-openclaw-token` header 提取
  3. 返回提取的令牌（去除空白）

### readJsonBody(req: IncomingMessage, maxBytes: number): Promise<...>
- **功能**: 读取和解析 JSON 请求体
- **参数**:
  - `req` - HTTP 请求对象
  - `maxBytes` - 最大字节数
- **返回值**: Promise，包含解析结果或错误
- **实现逻辑**:
  1. 使用 `readJsonBodyWithLimit` 读取请求体
  2. 处理各种错误情况：
     - `PAYLOAD_TOO_LARGE` → 返回 "payload too large"
     - `REQUEST_BODY_TIMEOUT` → 返回 "request body timeout"
     - `CONNECTION_CLOSED` → 返回连接关闭错误
  3. 返回解析结果或错误消息

### normalizeHookHeaders(req: IncomingMessage): Record<string, string>
- **功能**: 标准化 HTTP 请求头
- **参数**: HTTP 请求对象
- **返回值**: 标准化的请求头对象（键转为小写）
- **实现逻辑**:
  1. 遍历所有请求头
  2. 字符串值直接使用，数组值用逗号连接
  3. 所有键转为小写
  4. 返回标准化后的请求头对象

### normalizeWakePayload(payload: Record<string, unknown>): ...
- **功能**: 标准化唤醒请求负载
- **参数**: 请求负载对象
- **返回值**: 标准化结果或错误
- **实现逻辑**:
  1. 提取并验证 text 字段（必需）
  2. 解析 mode 字段（默认 "now"）
  3. 返回标准化后的对象

### resolveHookChannel(raw: unknown): HookMessageChannel | null
- **功能**: 解析 Webhook 通道标识
- **参数**: 原始通道值
- **返回值**: 通道标识或 null（无效）
- **实现逻辑**:
  1. 未定义时返回 "last"
  2. 必须是字符串
  3. 标准化通道名称
  4. 验证通道在允许列表中
  5. 返回有效的通道标识

### resolveHookDeliver(raw: unknown): boolean
- **功能**: 解析 deliver 标志
- **参数**: 原始值
- **返回值**: 布尔值（默认 true）
- **实现逻辑**: 只有显式 false 才返回 false

### resolveHookTargetAgentId(hooksConfig, agentId): string | undefined
- **功能**: 解析目标代理 ID
- **参数**:
  - `hooksConfig` - Webhook 配置
  - `agentId` - 原始代理 ID
- **返回值**: 代理 ID 或 undefined
- **实现逻辑**:
  1. 空值返回 undefined
  2. 标准化代理 ID
  3. 如果是已知代理，返回它
  4. 否则返回默认代理 ID

### isHookAgentAllowed(hooksConfig, agentId): boolean
- **功能**: 检查代理是否被允许
- **参数**:
  - `hooksConfig` - Webhook 配置
  - `agentId` - 代理 ID
- **返回值**: 是否允许
- **实现逻辑**:
  1. 空值允许（向后兼容）
  2. 无限制允许
  3. 解析目标代理并检查是否在允许列表中

### resolveHookSessionKey(params): ...
- **功能**: 解析和生成会话键
- **参数**:
  - `hooksConfig` - Webhook 配置
  - `source` - 来源类型
  - `sessionKey` - 请求的会话键
  - `idFactory` - ID 生成函数
- **返回值**: 解析结果或错误
- **实现逻辑**:
  1. 如果请求提供了会话键：
     - 检查来源是否允许（request 来源需要 allowRequestSessionKey=true）
     - 验证会话键前缀是否符合规则
     - 返回提供的会话键
  2. 使用默认会话键（如果有）
  3. 否则生成新会话键（格式: `hook:<uuid>`）
  4. 验证生成的会话键前缀
  5. 返回最终会话键

### normalizeAgentPayload(payload): ...
- **功能**: 标准化代理 Webhook 请求负载
- **参数**: 请求负载对象
- **返回值**: 标准化结果或错误
- **实现逻辑**:
  1. 验证必需字段（message）
  2. 解析可选字段（name, agentId, sessionKey, to, model, thinking）
  3. 解析和验证字段（wakeMode, channel, deliver, timeoutSeconds）
  4. 返回标准化的 HookAgentPayload 对象

## 主要依赖
- `node:http` - HTTP 请求处理
- `node:crypto` - UUID 生成
- `../config/config.js` - OpenClaw 配置
- `../agents/agent-scope.js` - 代理范围和 ID
- `../channels/plugins/index.js` - 通道插件列表
- `../infra/http-body.js` - HTTP 请求体读取
- `../routing/session-key.js` - 会话键标准化
- `../utils/message-channel.js` - 消息通道标准化
- `./hooks-mapping.js` - Webhook 映射解析

## 使用场景
- 外部 Webhook 接收
- HTTP API 集成
- 第三方服务触发
- 自动化工作流集成
- 外部系统与 OpenClaw 通信

## 代码行数
386 行

## 重要特性
- 强认证（Bearer token 或自定义 header）
- 请求体大小限制（防止 DoS）
- 会话策略控制（默认会话键、前缀限制、请求控制）
- 代理访问控制（白名单、通配符支持）
- 消息通道映射（支持所有通道 + "last"）
- 严格的输入验证和错误处理
- 灵活的配置策略

## 默认值
- **基础路径**: `/hooks`
- **最大请求体**: 256KB
- **默认通道**: `last`
- **默认名称**: `Hook`
- **默认唤醒模式**: `now`
- **默认 deliver**: `true`

## 认证方式
1. **Authorization header**: `Bearer <token>`
2. **自定义 header**: `x-openclaw-token: <token>`

## 会话键解析优先级
1. 请求提供的 `sessionKey`（如果允许）
2. 配置中的 `defaultSessionKey`
3. 自动生成（`hook:<uuid>`）

## 代理 ID 解析优先级
1. 请求的 `agentId`（如果在已知列表中）
2. 配置中的默认代理 ID

## 使用示例
```typescript
// 解析配置
const hooksConfig = resolveHooksConfig(config);
if (hooksConfig) {
  console.log(`Webhooks enabled at ${hooksConfig.basePath}`);
}

// 提取认证令牌
const token = extractHookToken(req);
if (token !== hooksConfig?.token) {
  return { status: 401, body: "Unauthorized" };
}

// 读取请求体
const bodyResult = await readJsonBody(req, hooksConfig.maxBodyBytes);
if (!bodyResult.ok) {
  return { status: 400, body: bodyResult.error };
}

// 标准化代理负载
const payloadResult = normalizeAgentPayload(bodyResult.value);
if (!payloadResult.ok) {
  return { status: 400, body: payloadResult.error };
}

// 解析会话键
const sessionKeyResult = resolveHookSessionKey({
  hooksConfig,
  source: "request",
  sessionKey: payloadResult.value.sessionKey,
  idFactory: () => customIdGenerator(),
});
```

## 配置示例
```typescript
{
  hooks: {
    enabled: true,
    token: "secret-webhook-token",
    path: "/webhooks",
    maxBodyBytes: 512_000, // 512KB
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:", "external:"],
    defaultSessionKey: "hook:main",
    allowedAgentIds: ["agent1", "agent2"], // 或 ["*"] 允许所有
  }
}
```

## 错误处理
- **配置错误**: 缺少 token、无效路径、策略冲突
- **认证错误**: token 不匹配或缺失
- **请求错误**: 请求体过大、无效 JSON、缺失必需字段
- **权限错误**: 代理不允许、会话键前缀不匹配

## 安全特性
- 强认证要求
- 请求体大小限制
- 严格的输入验证
- 会话隔离（前缀限制）
- 代理访问控制
- 拒绝通配符路径（`/`）
