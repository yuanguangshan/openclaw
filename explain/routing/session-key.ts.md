# routing/session-key.ts 解读

**文件路径**: `src/routing/session-key.ts`

## 文件用途
会话键构建、解析和规范化模块，负责生成和管理 OpenClaw 的会话标识符，支持多种会话类型和作用域。

## 主要常量

### 默认值
- `DEFAULT_AGENT_ID` - 默认代理 ID: `"main"`
- `DEFAULT_MAIN_KEY` - 默认主键: `"main"`
- `DEFAULT_ACCOUNT_ID` - 默认账户 ID: `"default"`

### 正则表达式
- `VALID_ID_RE` - 有效 ID 正则: `/^[a-z0-9][a-z0-9_-]{0,63}$/i`
  - 小写字母、数字、下划线、连字符
  - 最多 64 个字符
  - 必须以字母或数字开头

- `INVALID_CHARS_RE` - 无效字符正则: `/[^a-z0-9_-]+/g`
- `LEADING_DASH_RE` - 前导连字符正则: `/^-+/`
- `TRAILING_DASH_RE` - 尾部连字符正则: `/-+$/`

## 主要类型定义

### SessionKeyShape
会话键形状
- `"missing"` - 缺失
- `"agent"` - 代理格式
- `"legacy_or_alias"` - 遗留或别名格式
- `"malformed_agent"` - 格式错误的代理格式

## 主要函数

### normalizeMainKey(value): string
- **功能**: 规范化主键
- **参数**: 主键字符串
- **返回值**: 规范化后的主键（小写）或默认值
- **实现逻辑**:
  1. 去除空白
  2. 转小写
  3. 空字符串返回默认值

### toAgentRequestSessionKey(storeKey): string | undefined
- **功能**: 转换存储键为请求会话键
- **参数**: 存储键
- **返回值**: 请求会话键或 undefined
- **实现逻辑**:
  1. 解析代理会话键
  2. 提取 rest 部分
  3. 返回请求键

### toAgentStoreSessionKey(params): string
- **功能**: 转换请求参数为存储会话键
- **参数**:
  - `agentId` - 代理 ID
  - `requestKey` - 请求键
  - `mainKey` - 主键
- **返回值**: 存储会话键
- **实现逻辑**:
  1. 空键或 "main" → 构建主会话键
  2. "agent:" 前缀 → 直接使用
  3. "subagent:" 前缀 → 构建代理前缀
  4. 其他 → 添加 "agent:" 前缀

### resolveAgentIdFromSessionKey(sessionKey): string
- **功能**: 从会话键解析代理 ID
- **参数**: 会话键
- **返回值**: 代理 ID（规范化）
- **实现逻辑**:
  1. 解析代理会话键
  2. 提取代理 ID
  3. 规范化并返回

### classifySessionKeyShape(sessionKey): SessionKeyShape
- **功能**: 分类会话键形状
- **参数**: 会话键
- **返回值**: 会话键形状类型
- **实现逻辑**:
  1. 空字符串 → "missing"
  2. 有效代理格式 → "agent"
  3. 以 "agent:" 开头但无效 → "malformed_agent"
  4. 其他 → "legacy_or_alias"

### normalizeAgentId(value): string
- **功能**: 规范化代理 ID
- **参数**: 代理 ID
- **返回值**: 规范化的代理 ID
- **实现逻辑**:
  1. 空值返回默认值
  2. 有效格式 → 转小写
  3. 无效格式 → 替换无效字符、去除首尾连字符、限制长度

### sanitizeAgentId(value): string
- **功能**: 清理代理 ID（同 normalizeAgentId）
- **参数**: 代理 ID
- **返回值**: 清理后的代理 ID
- **实现逻辑**: 与 `normalizeAgentId` 相同

### normalizeAccountId(value): string
- **功能**: 规范化账户 ID
- **参数**: 账户 ID
- **返回值**: 规范化的账户 ID
- **实现逻辑**: 与 `normalizeAgentId` 相同，但使用账户默认值

### buildAgentMainSessionKey(params): string
- **功能**: 构建代理主会话键
- **参数**:
  - `agentId` - 代理 ID
  - `mainKey` - 主键
- **返回值**: 格式: `"agent:{agentId}:{mainKey}"`

### buildAgentPeerSessionKey(params): string
- **功能**: 构建代理对等体会话键
- **参数**:
  - `agentId` - 代理 ID
  - `mainKey` - 主键
  - `channel` - 通道 ID
  - `accountId` - 账户 ID
  - `peerKind` - 对等类型
  - `peerId` - 对等 ID
  - `identityLinks` - 身份链接
  - `dmScope` - DM 会话作用域
- **返回值**: 格式取决于作用域
- **实现逻辑**:
  1. DM 会话 → 根据 dmScope 构建
  2. 群组会话 → 构建群组键
  3. 支持身份链接解析

### buildGroupHistoryKey(params): string
- **功能**: 构建群组历史键
- **参数**:
  - `channel` - 通道 ID
  - `accountId` - 账户 ID
  - `peerKind` - 对等类型
  - `peerId` - 对等 ID
- **返回值**: 格式: `"{channel}:{accountId}:{peerKind}:{peerId}"`

### resolveThreadSessionKeys(params): ...
- **功能**: 解析线程会话键
- **参数**:
  - `baseSessionKey` - 基础会话键
  - `threadId` - 线程 ID
  - `parentSessionKey` - 父会话键
  - `useSuffix` - 是否使用后缀
- **返回值**: `{ sessionKey, parentSessionKey? }`
- **实现逻辑**:
  1. 无线程 ID → 返回基础键
  2. 有线程 ID → 添加线程后缀
  3. 返回会话键和父会话键

## 辅助函数

### resolveLinkedPeerId(params): string | null
- **功能**: 解析链接的对等 ID
- **参数**:
  - `identityLinks` - 身份链接映射
  - `channel` - 通道
  - `peerId` - 对等 ID
- **返回值**: 规范化名称或 null
- **实现逻辑**:
  1. 构建候选 ID（peerId 和 channel:peerId）
  2. 在身份链接中查找匹配
  3. 返回规范化名称

### normalizeToken(value): string
- **功能**: 规范化令牌
- **参数**: 任意字符串
- **返回值**: 规范化的令牌（小写，去空白）

## 主要依赖
- `../channels/chat-type.js` - 聊天类型
- `../sessions/session-key-utils.js` - 会话键工具

## 使用场景
- 会话键生成和解析
- ID 规范化和验证
- 会话路由和隔离
- 身份链接解析
- 线程会话管理

## 代码行数
263 行

## 重要特性
- 灵活的会话键格式
- ID 验证和规范化
- 多种会话作用域支持
- 身份链接解析
- 线程会话支持

## 会话键格式

### 主会话
```
agent:{agentId}:main
```

### DM 会话（各种作用域）
```
# main 作用域
agent:{agentId}:main

# per-peer 作用域
agent:{agentId}:direct:{peerId}

# per-channel-peer 作用域
agent:{agentId}:{channel}:direct:{peerId}

# per-account-channel-peer 作用域
agent:{agentId}:{channel}:{accountId}:direct:{peerId}
```

### 群组会话
```
agent:{agentId}:{channel}:group:{groupId}
agent:{agentId}:{channel}:channel:{channelId}
```

### 线程会话
```
{baseSessionKey}:thread:{threadId}
```

### 群组历史键
```
{channel}:{accountId}:{peerKind}:{peerId}
```

## 使用示例

### 规范化 ID
```typescript
normalizeAgentId("MyAgent");
// "myagent"

normalizeAgentId("invalid@name!");
// "invalid-name" (无效字符替换)

normalizeAccountId("");
// "default"

normalizeMainKey(null);
// "main"
```

### 构建会话键
```typescript
// 主会话
buildAgentMainSessionKey({ agentId: "myagent" });
// "agent:myagent:main"

// DM 会话
buildAgentPeerSessionKey({
  agentId: "myagent",
  channel: "whatsapp",
  peerKind: "direct",
  peerId: "1234567890"
});
// "agent:myagent:whatsapp:direct:1234567890"

// 群组会话
buildAgentPeerSessionKey({
  agentId: "myagent",
  channel: "telegram",
  peerKind: "group",
  peerId: "-1001234567890"
});
// "agent:myagent:telegram:group:-1001234567890"
```

### 解析会话键
```typescript
resolveAgentIdFromSessionKey("agent:myagent:main");
// "myagent"

resolveAgentIdFromSessionKey("agent:other:telegram:direct:123");
// "other"

resolveAgentIdFromSessionKey(null);
// "main"
```

### 分类会话键
```typescript
classifySessionKeyShape("agent:myagent:main");
// "agent"

classifySessionKeyShape("legacy-key");
// "legacy_or_alias"

classifySessionKeyShape("agent:invalid@format");
// "malformed_agent"

classifySessionKeyShape("");
// "missing"
```

### 转换会话键
```typescript
// 请求 → 存储
toAgentStoreSessionKey({
  agentId: "myagent",
  requestKey: "custom-session"
});
// "agent:myagent:custom-session"

// 存储 → 请求
toAgentRequestSessionKey("agent:myagent:custom-session");
// "custom-session"

toAgentRequestSessionKey("legacy-key");
// "legacy-key"
```

### 身份链接
```typescript
const links = {
  "canonical-user": ["user1", "user2", "telegram:user1"]
};

buildAgentPeerSessionKey({
  agentId: "myagent",
  channel: "telegram",
  peerKind: "direct",
  peerId: "user1",
  identityLinks: links,
  dmScope: "per-peer"
});
// "agent:myagent:direct:canonical-user" (链接到规范化名称)
```

### 线程会话
```typescript
resolveThreadSessionKeys({
  baseSessionKey: "agent:myagent:telegram:group:-100123",
  threadId: "456"
});
// {
//   sessionKey: "agent:myagent:telegram:group:-100123:thread:456",
//   parentSessionKey: "agent:myagent:telegram:group:-100123"
// }
```

### 群组历史键
```typescript
buildGroupHistoryKey({
  channel: "telegram",
  accountId: "bot1",
  peerKind: "group",
  peerId: "-1001234567890"
});
// "telegram:bot1:group:-1001234567890"
```

## DM 会话作用域

### main
所有 DM 消息使用主会话：
```
agent:{agentId}:main
```

### per-peer
每个对等用户独立会话：
```
agent:{agentId}:direct:{peerId}
```

### per-channel-peer
每个通道的每个对等用户独立会话：
```
agent:{agentId}:{channel}:direct:{peerId}
```

### per-account-channel-peer
每个账户的每个通道的每个对等用户独立会话：
```
agent:{agentId}:{channel}:{accountId}:direct:{peerId}
```

## ID 规范化规则

### 有效格式
- 小写字母
- 数字
- 下划线 `_`
- 连字符 `-`
- 1-64 个字符
- 以字母或数字开头

### 无效字符处理
- 无效字符替换为 `-`
- 移除前导和尾部 `-`
- 截断到 64 个字符
- 如果处理后为空，使用默认值

## 身份链接

### 用途
将不同平台的相同用户关联到统一的规范化名称。

### 格式
```typescript
{
  "canonical-name": ["alias1", "alias2", "channel:peerId"]
}
```

### 解析逻辑
1. 构建候选 ID 集合
2. 在身份链接中查找匹配
3. 返回规范化名称

## 线程会话

### 后缀格式
```
{baseSessionKey}:thread:{threadId}
```

### 用途
- 支持平台线程（如 Telegram 线程）
- 保持与父会话的关联
- 独立的线程历史

### 使用选项
- `useSuffix: false` - 不添加线程后缀（线程与主会话合并）
- `useSuffix: true` - 添加线程后缀（默认）

## 验证和分类

### 验证
- 使用 `VALID_ID_RE` 验证格式
- 确保路径安全和 shell 友好

### 分类
- 识别会话键类型
- 支持迁移和兼容性
- 检测格式错误

## 默认值

### 默认 ID
- `DEFAULT_AGENT_ID = "main"`
- `DEFAULT_MAIN_KEY = "main"`
- `DEFAULT_ACCOUNT_ID = "default"`

### 使用场景
- 未指定时使用
- 规范化失败时回退
- 保证有效的标识符

## 性能考虑
- 预编译正则表达式
- 早期返回（避免不必要的处理）
- 使用 Set 进行高效查找

## 安全考虑
- ID 验证防止注入
- 路径安全（无特殊字符）
- Shell 友好（无引号问题）

## 扩展性

### 添加新的作用域
```typescript
// 在 buildAgentPeerSessionKey 中添加
if (dmScope === "new-scope") {
  return `agent:${agentId}:${channel}:${accountId}:custom:${peerId}`;
}
```

### 自定义规范化
```typescript
// 修改 normalizeAgentId 逻辑
export function normalizeAgentId(value: string | undefined | null): string {
  // 自定义逻辑...
}
```

## 相关文件
- `../sessions/session-key-utils.js` - 会话键工具
- `../channels/chat-type.js` - 聊天类型
- `../config/group-policy.js` - 群组策略
