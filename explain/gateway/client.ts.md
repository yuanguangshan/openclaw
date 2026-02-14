# gateway/client.ts 解读

**文件路径**: `src/gateway/client.ts`

## 文件用途
网关 WebSocket 客户端实现，提供与 OpenClaw 网关的连接、认证、消息处理和重连功能。

## 主要类型定义

### GatewayClientOptions
网关客户端选项：
- `url`: WebSocket URL（默认 ws://127.0.0.1:18789）
- `connectDelayMs`: 连接延迟
- `tickWatchMinIntervalMs`: 心跳监控最小间隔
- `token`: 认证 Token
- `password`: 认证密码
- `instanceId`: 实例 ID
- `clientName`: 客户端名称
- `clientDisplayName`: 客户端显示名称
- `clientVersion`: 客户端版本
- `platform`: 平台标识
- `mode`: 客户端模式
- `role`: 角色
- `scopes`: 权限范围
- `caps`: 能力
- `commands`: 支持的命令
- `permissions`: 权限映射
- `pathEnv`: PATH 环境变量
- `deviceIdentity`: 设备身份
- `minProtocol/maxProtocol`: 协议版本范围
- `tlsFingerprint`: TLS 指纹
- 回调函数: `onEvent`, `onHelloOk`, `onConnectError`, `onClose`, `onGap`

### Pending
待处理的请求：
- `resolve`: 成功回调
- `reject`: 失败回调
- `expectFinal`: 是否等待最终响应

## 主要常量

### GATEWAY_CLOSE_CODE_HINTS
WebSocket 关闭代码提示：
- 1000: 正常关闭
- 1006: 异常关闭（无关闭帧）
- 1008: 策略违规
- 1012: 服务重启

## GatewayClient 类

### 主要属性
- `ws`: WebSocket 连接实例
- `opts`: 客户端配置
- `pending`: 待处理请求映射
- `backoffMs`: 重连退避时间
- `closed`: 是否已关闭
- `lastSeq`: 最后接收的序列号
- `connectNonce`: 连接随机数
- `connectSent`: 是否已发送连接请求
- `connectTimer`: 连接定时器
- `lastTick`: 最后心跳时间
- `tickIntervalMs`: 心跳间隔
- `tickTimer`: 心跳定时器

### 主要方法

#### constructor(opts: GatewayClientOptions)
- **功能**: 构造函数
- **参数**: 客户端选项
- **实现**: 加载或创建设备身份

#### start()
- **功能**: 启动客户端连接
- **实现逻辑**:
  1. 验证 TLS 指纹配置
  2. 创建 WebSocket 连接
  3. 设置最大负载（25MB）
  4. 配置 TLS 证书验证
  5. 设置事件处理器

#### stop()
- **功能**: 停止客户端
- **实现**: 清理定时器、关闭连接、清除待处理请求

#### sendConnect()
- **功能**: 发送连接请求
- **实现逻辑**:
  1. 加载设备 Token
  2. 构建设备认证签名
  3. 创建连接参数
  4. 发送连接请求
  5. 处理响应和错误

#### handleMessage(raw: string)
- **功能**: 处理接收到的消息
- **实现逻辑**:
  1. 解析 JSON
  2. 验证事件帧
  3. 处理连接挑战
  4. 检测消息间隙
  5. 处理心跳
  6. 处理响应帧

#### queueConnect()
- **功能**: 队列化连接请求
- **实现**: 延迟发送连接（默认 750ms）

#### scheduleReconnect()
- **功能**: 安排重连
- **实现**: 指数退避，最大 30 秒

#### flushPendingErrors(err: Error)
- **功能**: 刷新待处理错误
- **实现**: 拒绝所有待处理请求

#### startTickWatch()
- **功能**: 启动心跳监控
- **实现**: 检查心跳超时，超时则关闭连接

#### validateTlsFingerprint(): Error | null
- **功能**: 验证 TLS 指纹
- **返回值**: 验证错误或 null

#### request<T>(method, params?, opts?): Promise<T>
- **功能**: 发送请求到网关
- **参数**:
  - `method`: 方法名
  - `params`: 参数
  - `opts`: 选项（expectFinal）
- **返回值**: 响应 Promise

## 主要依赖
- `node:crypto` - UUID 生成
- `ws` - WebSocket 实现
- `../infra/device-identity.js` - 设备身份管理
- `../infra/tls/fingerprint.js` - TLS 指纹处理
- `../infra/ws.js` - WebSocket 工具
- `./device-auth.js` - 设备认证
- `./protocol/index.js` - 协议定义和验证

## 使用场景
- 与网关的 WebSocket 通信
- 设备身份认证
- 自动重连和错误恢复
- 心跳保活
- TLS 安全连接

## 代码行数
454 行

## 重要特性
- 自动重连和退避
- TLS 指纹验证
- 设备身份认证
- 心跳监控
- 消息间隙检测
- 大负载支持（25MB）
- 序列号追踪

## 使用示例
```typescript
const client = new GatewayClient({
  url: "ws://127.0.0.1:18789",
  token: "your-token",
  clientName: GATEWAY_CLIENT_NAMES.WEBCHAT_UI,
  onEvent: (evt) => console.log("Event:", evt),
  onHelloOk: (hello) => console.log("Connected:", hello),
  onClose: (code, reason) => console.log("Closed:", code, reason)
});

client.start();

// 发送请求
const result = await client.request("chat.send", {
  text: "Hello, OpenClaw!"
});
```