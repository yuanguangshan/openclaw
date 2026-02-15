# gateway/session-utils.ts 解读

**文件路径**: `src/gateway/session-utils.ts`

## 文件用途
网关会话工具模块，提供会话加载、解析、列表和管理功能，支持代理配置、身份头像和会话标题推导。

## 主要常量

### 限制和常量
- `DERIVED_TITLE_MAX_LEN` - 推导标题最大长度: 60
- `AVATAR_MAX_BYTES` - 头像最大字节数: 2MB

### 正则表达式
- `AVATAR_DATA_RE` - Data URL 正则: `/^data:/i`
- `AVATAR_HTTP_RE` - HTTP/HTTPS URL 正则: `/^https?:\/\//i`
- `AVATAR_SCHEME_RE` - URL scheme 正则: `/^[a-z][a-z0-9+.-]*:/i`
- `WINDOWS_ABS_RE` - Windows 绝对路径正则: `/^[a-zA-Z]:[\\/]/`

### MIME 类型映射
支持的图像格式及其 MIME 类型：
- `.png`: `image/png`
- `.jpg/.jpeg`: `image/jpeg`
- `.webp`: `image/webp`
- `.gif`: `image/gif`
- `.svg`: `image/svg+xml`
- `.bmp`: `image/bmp`
- `.tif/.tiff`: `image/tiff`

## 主要函数

### resolveIdentityAvatarUrl(cfg, agentId, avatar): string | undefined
- **功能**: 解析代理身份头像 URL
- **参数**:
  - `cfg` - 配置
  - `agentId` - 代理 ID
  - `avatar` - 头像路径或 URL
- **返回值**: Data URL（本地文件）或原始 URL
- **实现逻辑**:
  1. 检查是否为 data 或 http URL → 直接返回
  2. 验证是否为工作区相对路径
  3. 读取文件并转换为 base64 Data URL
  4. 限制文件大小为 2MB
  5. 失败时返回 undefined

### loadSessionEntry(sessionKey): ...
- **功能**: 加载会话条目
- **参数**: 会话键
- **返回值**: `{ cfg, storePath, store, entry, canonicalKey, legacyKey }`
- **实现逻辑**:
  1. 加载配置
  2. 解析存储键
  3. 加载存储
  4. 查找匹配（精确 + 大小写不敏感）
  5. 返回条目和键信息

### findStoreMatch(store, ...candidates): ...
- **功能**: 查找存储条目
- **参数**: 存储对象，候选键列表
- **返回值**: `{ entry, key }` 或 undefined
- **实现逻辑**:
  1. 精确匹配候选键
  2. 大小写不敏感扫描所有候选
  3. 返回第一个匹配

### findStoreKeysIgnoreCase(store, targetKey): string[]
- **功能**: 查找所有不区分大小写的匹配键
- **参数**: 存储对象，目标键
- **返回值**: 匹配的键数组
- **实现逻辑**: 遍历所有键，比较小写形式

### pruneLegacyStoreKeys(params): void
- **功能**: 移除遗留键变体
- **参数**: `{ store, canonicalKey, candidates }`
- **实现逻辑**:
  1. 收集所有非规范键
  2. 包含不区分大小写的匹配
  3. 从存储中删除

### classifySessionKey(key, entry?): "global" | "unknown" | "group" | "direct"
- **功能**: 分类会话键类型
- **参数**: 会话键，条目
- **返回值**: 会话类型
- **实现逻辑**:
  1. 特殊键："global", "unknown"
  2. 群组/通道：基于 chatType 或键格式
  3. 其他：direct

### parseGroupKey(key): ...
- **功能**: 解析群组键
- **参数**: 会话键
- **返回值**: `{ channel?, kind?, id? }` 或 null
- **实现逻辑**: 解析 `{channel}:{kind}:{id}` 格式

### listAgentsForGateway(cfg): ...
- **功能**: 列出网关的代理
- **参数**: 配置对象
- **返回值**: `{ defaultId, mainKey, scope, agents[] }`
- **实现逻辑**:
  1. 解析默认代理 ID 和主键
  2. 收集代理配置
  3. 解析身份信息（名称、主题、表情、头像）
  4. 应用允许列表过滤
  5. 确保主键在列表中

### resolveSessionStoreKey(params): string
- **功能**: 解析会话存储键
- **参数**: `{ cfg, sessionKey }`
- **返回值**: 规范化的存储键
- **实现逻辑**:
  1. 特殊键直接返回
  2. 解析代理会话键
  3. 处理主会话键别名
  4. 添加代理前缀（如需要）

### resolveSessionStoreAgentId(cfg, canonicalKey): string
- **功能**: 解析会话存储的代理 ID
- **返回值**: 代理 ID（规范化）

### canonicalizeSpawnedByForAgent(cfg, agentId, spawnedBy?): ...
- **功能**: 规范化生成者引用
- **参数**: 配置、代理 ID、原始生成者
- **返回值**: 规范化的生成者键或 undefined
- **实现逻辑**: 解析代理前缀并处理主会话键别名

### resolveGatewaySessionStoreTarget(params): ...
- **功能**: 解析网关会话存储目标
- **参数**: `{ cfg, key, scanLegacyKeys?, store? }`
- **返回值**: `{ agentId, storePath, canonicalKey, storeKeys[] }`
- **实现逻辑**:
  1. 解析规范键
  2. 解析代理 ID 和存储路径
  3. 扫描遗留键变体
  4. 返回所有匹配的键

### listSessionsFromStore(params): ...
- **功能**: 从存储列出会话
- **参数**: `{ cfg, storePath, store, opts }`
- **返回值**: 会话列表结果
- **实现逻辑**:
  1. 根据选项过滤会话
  2. 解析会话元数据
  3. 从转录读取标题和最后消息
  4. 搜索和活跃时间过滤
  5. 按更新时间排序

### getSessionDefaults(cfg): ...
- **功能**: 获取会话默认值
- **返回值**: `{ modelProvider, model, contextTokens }`
- **实现逻辑**:
  1. 解析配置的模型引用
  2. 查找上下文令牌数
  3. 返回默认值

### resolveSessionModelRef(cfg, entry?, agentId?): ...
- **功能**: 解析会话模型引用
- **返回值**: `{ provider, model }`
- **实现逻辑**:
  1. 解析代理或全局默认模型
  2. 应用存储的模型覆盖
  3. 返回提供商和模型

## 辅助函数

### deriveSessionTitle(entry, firstUserMessage?): string | undefined
- **功能**: 推导会话标题
- **返回值**: 标题字符串或 undefined
- **优先级**: displayName > subject > firstUserMessage > sessionId

### formatSessionIdPrefix(sessionId, updatedAt?): string
- **功能**: 格式化会话 ID 前缀
- **返回值**: `prefix (date)` 或 `prefix`

### truncateTitle(text, maxLen): string
- **功能**: 截断标题
- **返回值**: 截断后的文本（末尾添加 "…"）

### resolveAvatarMime(filePath): string
- **功能**: 解析头像 MIME 类型
- **实现逻辑**: 根据文件扩展名返回 MIME 类型

### isWorkspaceRelativePath(value): boolean
- **功能**: 检查是否为工作区相对路径
- **实现逻辑**: 拒绝 `~` 开头和 URL scheme

## 主要依赖
- `node:fs` - 文件系统操作
- `node:path` - 路径处理
- `../agents/agent-scope.js` - 代理范围
- `../agents/context.js` - 上下文令牌
- `../agents/defaults.js` - 默认值
- `../agents/model-selection.js` - 模型选择
- `../config/config.js` - 配置
- `../config/paths.js` - 路径
- `../config/sessions.js` - 会话配置
- `../routing/session-key.js` - 会话键
- `../sessions/session-key-utils.js` - 会话键工具
- `../utils/delivery-context.js` - 投递上下文
- `./session-utils.fs.js` - 文件系统操作

## 使用场景
- 会话管理
- 代理身份配置
- 会话列表和搜索
- 头像处理
- 标题推导
- 遗留键清理

## 代码行数
846 行

## 重要特性
- 灵活的会话键解析
- 大小写不敏感匹配
- 遗留键清理
- 头像 Data URL 转换
- 会话标题推导
- 多存储文件支持

## 会话键分类

### 分类规则
- `"global"` - 全局会话
- `"unknown"` - 未知来源会话
- `"group"` - 群组/通道会话
- `"direct"` - 直接消息会话

## 头像处理

### 支持格式
- **Data URL**: `data:image/png;base64,...`
- **HTTP/HTTPS URL**: `https://example.com/avatar.png`
- **工作区相对路径**: `avatar.png`, `images/avatar.png`

### 限制
- 最大 2MB
- 必须在工作区内
- 支持的图像格式

## 使用示例

### 加载会话条目
```typescript
const { entry, canonicalKey, legacyKey } = loadSessionEntry("agent:myagent:main");
if (legacyKey) {
  // 清理遗留键
  pruneLegacyStoreKeys({
    store,
    canonicalKey,
    candidates: [legacyKey]
  });
}
```

### 列出会话
```typescript
const result = listSessionsFromStore({
  cfg,
  storePath,
  store,
  opts: {
    includeGlobal: false,
    includeDerivedTitles: true,
    agentId: "myagent",
    activeMinutes: 60,
    limit: 20
  }
});
// { ts, path, count, defaults, sessions: [...] }
```

### 解析头像
```typescript
const avatarUrl = resolveIdentityAvatarUrl(cfg, "myagent", "avatar.png");
// "data:image/png;base64,iVBORw0KG..."
```

### 分类会话
```typescript
classifySessionKey("agent:myagent:telegram:group:-100123");
// "group"

classifySessionKey("agent:myagent:main");
// "direct"

classifySessionKey("global");
// "global"
```

### 解析群组键
```typescript
parseGroupKey("agent:myagent:telegram:group:-100123");
// { channel: "telegram", kind: "group", id: "-100123" }
```

### 规范化会话键
```typescript
resolveSessionStoreKey({
  cfg,
  sessionKey: "my-session"
});
// "agent:myagent:my-session"

resolveSessionStoreKey({
  cfg,
  sessionKey: "main"
});
// "agent:myagent:main"
```

### 列出代理
```typescript
const { defaultId, mainKey, scope, agents } = listAgentsForGateway(cfg);
// {
//   defaultId: "myagent",
//   mainKey: "main",
//   scope: "per-sender",
//   agents: [
//     { id: "myagent", name: "My Agent", identity: {...} },
//     ...
//   ]
// }
```
