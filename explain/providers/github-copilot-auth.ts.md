# providers/github-copilot-auth.ts 解读

**文件路径**: `src/providers/github-copilot-auth.ts`

## 文件用途
GitHub Copilot OAuth 登录命令实现，通过 GitHub 设备授权流程获取访问令牌。

## 主要类型定义

### DeviceCodeResponse
设备码响应
- `device_code: string` - 设备码
- `user_code: string` - 用户码
- `verification_uri: string` - 验证 URI
- `expires_in: number` - 过期时间（秒）
- `interval: number` - 轮询间隔（秒）

### DeviceTokenResponse
设备令牌响应
- **成功**: `{ access_token, token_type, scope? }`
- **失败**: `{ error, error_description?, error_uri? }`

## 主要函数

### githubCopilotLoginCommand(opts, runtime): Promise<void>
- **功能**: 执行 GitHub Copilot 登录命令
- **参数**:
  - `opts.profileId?: string` - 配置文件 ID
  - `opts.yes?: boolean` - 自动确认
  - `runtime` - 运行时环境
- **实现逻辑**:
  1. 检查 TTY 是否可用
  2. 确保认证配置文件存储已初始化
  3. 提示现有凭据（如果有）
  4. 请求 GitHub 设备码
  5. 显示验证 URI 和用户码
  6. 轮询等待用户授权
  7. 保存访问令牌到认证配置
  8. 更新 OpenClaw 配置

### requestDeviceCode(params): Promise<DeviceCodeResponse>
- **功能**: 请求 GitHub 设备码
- **参数**: `{ scope: string }` - OAuth 范围
- **返回值**: 设备码响应
- **实现逻辑**:
  1. 构建 POST 请求
  2. 发送到 GitHub 设备码端点
  3. 验证响应和必需字段
  4. 返回设备码信息

### pollForAccessToken(params): Promise<string>
- **功能**: 轮询获取访问令牌
- **参数**:
  - `deviceCode` - 设备码
  - `intervalMs` - 轮询间隔
  - `expiresAt` - 过期时间戳
- **返回值**: 访问令牌
- **实现逻辑**:
  1. 在过期前持续轮询
  2. 发送设备码请求令牌
  3. 处理各种响应状态：
     - `authorization_pending` - 继续等待
     - `slow_down` - 增加间隔
     - `expired_token` - 过期错误
     - `access_denied` - 用户拒绝
  4. 成功时返回访问令牌

## 主要常量

### GitHub OAuth 端点
- `CLIENT_ID` - GitHub OAuth 客户端 ID
- `DEVICE_CODE_URL` - 设备码端点
- `ACCESS_TOKEN_URL` - 令牌端点

## 主要依赖
- `@clack/prompts` - CLI 交互界面（intro、note、outro、spinner）
- `../runtime.js` - 运行时环境
- `../agents/auth-profiles.js` - 认证配置文件管理
- `../commands/models/shared.js` - 配置更新工具
- `../commands/onboard-auth.js` - 认证配置应用
- `../config/logging.js` - 配置日志
- `../terminal/prompt-style.js` - 提示样式

## 使用场景
- GitHub Copilot 集成
- OAuth 设备授权流程
- 认证配置文件管理
- 交互式 CLI 登录

## 代码行数
185 行

## 重要特性
- 设备授权流程（无密钥交互）
- 轮询等待用户授权
- 自动处理各种错误状态
- 集成认证配置文件
- 友好的 CLI 提示界面
- TTY 检查确保交互式环境

## OAuth 设备流程

### 步骤
1. **请求设备码**: 客户端向 GitHub 请求设备码
2. **用户授权**: 用户访问验证 URI 并输入用户码
3. **轮询令牌**: 客户端轮询等待授权完成
4. **获取令牌**: 授权成功后返回访问令牌

### 流程图
```
Client → GitHub: 请求设备码
GitHub → Client: device_code, user_code, verification_uri
Client → User: 访问 verification_uri，输入 user_code
User → GitHub: 授权
Client → GitHub: 轮询（使用 device_code）
GitHub → Client: access_token
```

## 错误处理

### 设备码错误
- HTTP 请求失败
- 响应缺少必需字段

### 轮询错误
- `authorization_pending` - 等待中，继续轮询
- `slow_down` - 减慢轮询
- `expired_token` - 设备码过期
- `access_denied` - 用户拒绝授权
- `unknown` - 未知错误

### 配置错误
- 现有配置文件提示
- TTY 不可用错误

## 默认行为
- **配置文件 ID**: `github-copilot:github`
- **OAuth 范围**: `read:user`
- **轮询间隔**: 至少 1 秒
- **令牌过期**: 设备码过期后失败

## 使用示例

### 基本登录
```typescript
await githubCopilotLoginCommand({}, runtime);
// 交互式提示用户授权
```

### 指定配置文件
```typescript
await githubCopilotLoginCommand(
  { profileId: "my-github-profile" },
  runtime
);
```

### 覆盖现有配置
```typescript
await githubCopilotLoginCommand(
  { profileId: "github-copilot:github", yes: true },
  runtime
);
// 不提示，直接覆盖
```

## CLI 交互流程

### 输出示例
```
GitHub Copilot login
────────────────────

⠋ Requesting device code from GitHub...
✔ Device code ready

Authorize
─────────
Visit: https://github.com/login/device/code
Code: ABCD-1234

⠋ Waiting for GitHub authorization...
✔ GitHub access token acquired

Config updated.
Auth profile: github-copilot:github (github-copilot/token)

Done
```

## 配置文件结构

### 认证配置文件
```typescript
{
  profiles: {
    "github-copilot:github": {
      type: "token",
      provider: "github-copilot",
      token: "ghp_xxxxxxxxxxxx",
      // expires 未设置，将在 Copilot 令牌交换时设置
    }
  }
}
```

### OpenClaw 配置
```typescript
{
  agents: {
    defaults: {
      authProfiles: ["github-copilot:github"]
    }
  }
}
```

## 安全考虑

### 客户端 ID
- 硬编码客户端 ID（公开信息）
- 用于 OAuth 识别

### 令牌存储
- 存储在认证配置文件中
- 使用密钥链（如果可用）
- 令牌不记录到日志

### 传输安全
- 使用 HTTPS
- OAuth 标准流程

## 轮询策略

### 间隔处理
- 使用 GitHub 返回的间隔
- 最小 1 秒
- `slow_down` 时增加 2 秒

### 过期处理
- 跟踪过期时间
- 超时后停止轮询
- 抛出过期错误

## TTY 检查

### 为什么需要 TTY
- 交互式提示
- 用户输入确认
- 实时状态更新

### 错误消息
```
github-copilot login requires an interactive TTY.
```

## 扩展性

### 添加新提供商
复制此模式：
1. 定义 OAuth 端点
2. 实现设备码请求
3. 实现轮询逻辑
4. 保存令牌到配置
5. 更新 OpenClaw 配置

### 自定义范围
```typescript
const device = await requestDeviceCode({ 
  scope: "read:user repo" 
});
```

## 故障排除

### 常见问题
1. **授权超时**: 设备码过期，重新运行登录
2. **授权拒绝**: 用户取消，重新运行登录
3. **网络错误**: 检查网络连接
4. **TTY 错误**: 确保在交互式终端中运行

### 调试
- 检查 `device_code` 和 `user_code`
- 确认 `verification_uri` 正确
- 验证轮询间隔和过期时间

## 相关文件
- `github-copilot-token.ts` - Copilot 令牌交换
- `github-copilot-models.ts` - Copilot 模型定义
- `onboard-auth.ts` - 通用认证流程
- `auth-profiles.ts` - 认证配置文件管理
