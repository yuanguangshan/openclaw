# gateway/call.ts 解读

**文件路径**: `src/gateway/call.ts`

## 文件用途
网关调用核心模块，负责建立与 OpenClaw Gateway 的 WebSocket 连接，处理认证、超时、错误处理，并执行远程方法调用。

## 主要类型定义

### CallGatewayOptions
调用网关的配置选项
- `url?: string` - 网关 URL 覆盖
- `token?: string` - 认证令牌
- `password?: string` - 认证密码
- `tlsFingerprint?: string` - TLS 指纹
- `config?: OpenClawConfig` - 配置对象
- `method: string` - 要调用的方法名
- `params?: unknown` - 方法参数
- `expectFinal?: boolean` - 是否期望最终响应
- `timeoutMs?: number` - 超时时间（毫秒）
- `clientName?: GatewayClientName` - 客户端名称
- `clientDisplayName?: string` - 客户端显示名称
- `clientVersion?: string` - 客户端版本
- `platform?: string` - 平台信息
- `mode?: GatewayClientMode` - 客户端模式
- `instanceId?: string` - 实例 ID
- `minProtocol?: number` - 最小协议版本
- `maxProtocol?: number` - 最大协议版本
- `configPath?: string` - 配置文件路径（仅用于错误信息）

### GatewayConnectionDetails
网关连接详情
- `url: string` - 连接 URL
- `urlSource: string` - URL 来源说明
- `bindDetail?: string` - 绑定详情
- `remoteFallbackNote?: string` - 远程回退提示
- `message: string` - 完整的连接信息消息

### ExplicitGatewayAuth
显式网关认证
- `token?: string` - 认证令牌
- `password?: string` - 认证密码

## 主要函数

### resolveExplicitGatewayAuth(opts?: ExplicitGatewayAuth): ExplicitGatewayAuth
- **功能**: 解析和标准化显式网关认证凭据
- **参数**: 认证选项对象
- **返回值**: 标准化的认证对象（去除空白）
- **实现逻辑**:
  1. 清理 token 和 password 字符串
  2. 过滤空字符串和无效值
  3. 返回有效的认证凭据

### ensureExplicitGatewayAuth(params): void
- **功能**: 确保 URL 覆盖时必须提供显式认证
- **参数**:
  - `urlOverride?: string` - URL 覆盖
  - `auth: ExplicitGatewayAuth` - 认证对象
  - `errorHint: string` - 错误提示
  - `configPath?: string` - 配置路径
- **返回值**: 无
- **实现逻辑**:
  1. 检查是否有 URL 覆盖
  2. 检查是否有认证凭据（token 或 password）
  3. 如果缺少认证，抛出错误并提供修复建议
  4. 包含配置路径信息用于调试

### buildGatewayConnectionDetails(options): GatewayConnectionDetails
- **功能**: 构建网关连接详细信息
- **参数**:
  - `config?: OpenClawConfig` - 配置对象
  - `url?: string` - URL 覆盖
  - `configPath?: string` - 配置路径
- **返回值**: 连接详情对象
- **实现逻辑**:
  1. 加载配置或使用提供的配置
  2. 检查是否为远程模式
  3. 确定绑定模式（loopback/lan/tailnet）
  4. 根据 TLS 设置选择协议（ws/wss）
  5. 构建本地 URL（优先 tailnet，其次 lan，最后 loopback）
  6. 处理 URL 覆盖和远程 URL
  7. 检测远程模式配置错误
  8. 生成详细的连接信息消息

### callGateway<T>(opts: CallGatewayOptions): Promise<T>
- **功能**: 调用网关方法（核心函数）
- **参数**: 调用选项对象
- **返回值**: Promise<T> - 方法调用结果
- **实现逻辑**:
  1. 设置超时时间（默认 10 秒）
  2. 加载配置
  3. 解析认证凭据（优先级：显式 > 配置 > 环境变量）
  4. 验证远程模式配置
  5. 构建连接详情
  6. 处理 TLS 配置和指纹
  7. 创建 GatewayClient 实例
  8. 在 onHelloOk 回调中执行请求
  9. 设置超时定时器
  10. 处理连接关闭和超时错误
  11. 启动客户端连接

### randomIdempotencyKey(): string
- **功能**: 生成随机幂等性键
- **参数**: 无
- **返回值**: UUID 字符串
- **实现逻辑**: 使用 randomUUID() 生成唯一标识符

## 主要依赖
- `node:crypto` - UUID 生成
- `../config/config.js` - 配置加载和解析
- `../infra/device-identity.js` - 设备身份
- `../infra/tailnet.js` - Tailscale 网络处理
- `../infra/tls/gateway.js` - TLS 运行时
- `./client.js` - GatewayClient 类
- `./net.js` - 网络工具（LAN IPv4）
- `./protocol/index.js` - 协议版本

## 使用场景
- CLI 命令执行（如 `openclaw agent`, `openclaw message send`）
- 网关远程方法调用
- 多种连接模式（本地/远程/Tailnet）
- TLS 安全连接
- 超时和错误处理

## 代码行数
313 行

## 重要特性
- 多层认证支持（显式、配置、环境变量）
- 灵活的连接模式（loopback/lan/tailnet/remote）
- TLS 指纹验证
- 超时保护和优雅错误处理
- 详细的连接信息用于调试
- 协议版本协商
- 设备身份集成

## URL 解析优先级
1. CLI `--url` 覆盖（最高）
2. `gateway.remote.url`（远程模式）
3. 本地 tailnet 地址（如果 `bind=tailnet`）
4. 本地 LAN 地址（如果 `bind=lan`）
5. 本地 loopback（默认）

## 认证优先级
### Token
1. 显式 `opts.token`
2. `gateway.remote.token`（远程模式）
3. 环境变量 `OPENCLAW_GATEWAY_TOKEN`
4. `gateway.auth.token`

### Password
1. 显式 `opts.password`
2. 环境变量 `OPENCLAW_GATEWAY_PASSWORD`
3. `gateway.remote.password`（远程模式）
4. `gateway.auth.password`

## 错误处理
- **远程模式配置错误**: 当 `mode=remote` 但缺少 `remote.url` 时
- **认证缺失**: URL 覆盖时必须提供认证
- **连接关闭**: 格式化关闭码和原因
- **超时**: 超时后自动关闭连接

## 使用示例
```typescript
// 基本调用
const result = await callGateway({
  method: "chat.send",
  params: { message: "Hello" },
});

// 带超时的调用
const result = await callGateway({
  method: "agent.run",
  params: { task: "Analyze data" },
  timeoutMs: 30_000, // 30 秒超时
});

// 远程网关调用
const result = await callGateway({
  url: "wss://remote-gateway.example.com:18789",
  token: "secret-token",
  method: "sessions.list",
});

// 带客户端信息的调用
const result = await callGateway({
  method: "gateway.status",
  clientName: "my-app",
  clientDisplayName: "My Application",
  clientVersion: "1.0.0",
  platform: "darwin",
});
```

## 连接流程
```
1. 解析配置和认证
2. 构建连接 URL（考虑 bind 模式和远程配置）
3. 设置 TLS（如果启用）
4. 创建 GatewayClient 实例
5. 启动 WebSocket 连接
6. 等待 hello 握手
7. 发送请求并等待响应
8. 处理响应或错误
9. 关闭连接
```
