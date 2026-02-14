# gateway/auth.ts 解读

**文件路径**: `src/gateway/auth.ts`

## 文件用途
网关认证和授权模块，处理多种认证方式（token、密码、Tailscale、受信任代理），提供请求验证和安全访问控制。

## 主要类型定义

### ResolvedGatewayAuthMode
解析后的网关认证模式：
- `none` - 无认证
- `token` - Token 认证
- `password` - 密码认证
- `trusted-proxy` - 受信任代理认证

### ResolvedGatewayAuth
解析后的网关认证配置：
- `mode`: 认证模式
- `token`: Token 值
- `password`: 密码值
- `allowTailscale`: 是否允许 Tailscale 认证
- `trustedProxy`: 受信任代理配置

### GatewayAuthResult
网关认证结果：
- `ok`: 是否成功
- `method`: 认证方法
- `user`: 用户标识
- `reason`: 失败原因
- `rateLimited`: 是否被速率限制
- `retryAfterMs`: 重试等待时间（毫秒）

### ConnectAuth
连接认证信息：
- `token`: Token 值
- `password`: 密码值

### TailscaleUser
Tailscale 用户信息：
- `login`: 登录标识
- `name`: 显示名称
- `profilePic`: 头像 URL

## 主要函数

### normalizeLogin(login: string): string
- **功能**: 标准化登录名
- **参数**: 登录字符串
- **返回值**: 标准化后的登录名（小写、去空白）

### getHostName(hostHeader?: string): string
- **功能**: 从 Host 头提取主机名
- **参数**: Host 头值
- **返回值**: 主机名（去除端口号）

### headerValue(value: string | string[] | undefined): string | undefined
- **功能**: 标准化 HTTP 头值
- **参数**: 头值
- **返回值**: 标准化后的字符串

### resolveTailscaleClientIp(req?: IncomingMessage): string | undefined
- **功能**: 从请求解析 Tailscale 客户端 IP
- **参数**: HTTP 请求对象
- **返回值**: 客户端 IP

### resolveRequestClientIp(req?, trustedProxies?): string | undefined
- **功能**: 解析请求的客户端 IP（考虑受信任代理）
- **参数**:
  - `req`: HTTP 请求对象
  - `trustedProxies`: 受信任代理列表
- **返回值**: 客户端 IP

### isLocalDirectRequest(req?, trustedProxies?): boolean
- **功能**: 判断是否为本地直接请求
- **参数**:
  - `req`: HTTP 请求对象
  - `trustedProxies`: 受信任代理列表
- **返回值**: 布尔值
- **检查**:
  - 客户端 IP 是否为环回地址
  - 主机名是否为 localhost 或 Tailscale Serve
  - 无转发头或来自受信任代理

### getTailscaleUser(req?: IncomingMessage): TailscaleUser | null
- **功能**: 从请求头提取 Tailscale 用户信息
- **参数**: HTTP 请求对象
- **返回值**: Tailscale 用户或 null

### hasTailscaleProxyHeaders(req?: IncomingMessage): boolean
- **功能**: 检查是否有 Tailscale 代理头
- **参数**: HTTP 请求对象
- **返回值**: 布尔值

### isTailscaleProxyRequest(req?: IncomingMessage): boolean
- **功能**: 判断是否为 Tailscale 代理请求
- **参数**: HTTP 请求对象
- **返回值**: 布尔值

### resolveVerifiedTailscaleUser(params): Promise<...>
- **功能**: 验证并解析 Tailscale 用户
- **参数**:
  - `req`: HTTP 请求对象
  - `tailscaleWhois`: Tailscale whois 查询函数
- **返回值**: 验证结果
- **验证逻辑**:
  1. 检查 Tailscale 用户头
  2. 验证代理请求
  3. 查询 whois 信息
  4. 比对登录名

### resolveGatewayAuth(params): ResolvedGatewayAuth
- **功能**: 解析网关认证配置
- **参数**:
  - `authConfig`: 认证配置
  - `env`: 环境变量
  - `tailscaleMode`: Tailscale 模式
- **返回值**: 解析后的认证配置

### assertGatewayAuthConfigured(auth: ResolvedGatewayAuth): void
- **功能**: 断言网关认证已正确配置
- **参数**: 解析后的认证配置
- **抛出**: 配置不完整时抛出错误

### authorizeTrustedProxy(params): { user: string } | { reason: string }
- **功能**: 授权受信任代理请求
- **参数**:
  - `req`: HTTP 请求对象
  - `trustedProxies`: 受信任代理列表
  - `trustedProxyConfig`: 受信任代理配置
- **返回值**: 用户信息或失败原因

### authorizeGatewayConnect(params): Promise<GatewayAuthResult>
- **功能**: 授权网关连接
- **参数**:
  - `auth`: 认证配置
  - `connectAuth`: 连接认证信息
  - `req`: HTTP 请求对象
  - `trustedProxies`: 受信任代理列表
  - `tailscaleWhois`: Tailscale whois 查询函数
  - `rateLimiter`: 速率限制器（可选）
  - `clientIp`: 客户端 IP
  - `rateLimitScope`: 速率限制范围
- **返回值**: 认证结果

## 认证流程
1. 检查受信任代理模式
2. 检查速率限制
3. 尝试 Tailscale 认证（如果启用）
4. 尝试 Token 认证
5. 尝试密码认证
6. 返回失败结果

## 主要依赖
- `node:http` - HTTP 请求/响应
- `../config/config.js` - 配置类型
- `../infra/tailscale.js` - Tailscale 集成
- `../security/secret-equal.js` - 安全密钥比较
- `./auth-rate-limit.js` - 速率限制
- `./net.js` - 网络工具

## 使用场景
- 网关安全访问控制
- 多种认证方式支持
- 速率限制和防滥用
- Tailscale VPN 集成

## 代码行数
401 行

## 重要特性
- 多种认证方式
- 速率限制支持
- Tailscale 集成
- 受信任代理支持
- 安全密钥比较