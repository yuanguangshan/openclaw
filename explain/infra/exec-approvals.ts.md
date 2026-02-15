# infra/exec-approvals.ts 解读

**文件路径**: `src/infra/exec-approvals.ts`

## 文件用途
执行批准配置和管理模块，处理命令执行的批准流程、白名单管理和安全策略。

## 主要类型定义

### ExecHost
执行主机类型
- `"sandbox"` - 沙盒环境
- `"gateway"` - 网关主机
- `"node"` - Node 环境

### ExecSecurity
安全策略
- `"deny"` - 拒绝
- `"allowlist"` - 白名单
- `"full"` - 完全访问

### ExecAsk
询问策略
- `"off"` - 不询问
- `"on-miss"` - 白名单缺失时询问
- `"always"` - 总是询问

### ExecApprovalsFile
执行批准配置文件
- `version: 1` - 版本号
- `socket` - Socket 配置（path, token）
- `defaults` - 默认策略
- `agents` - 代理特定策略

### ExecApprovalsSnapshot
配置快照
- `path` - 文件路径
- `exists` - 是否存在
- `raw` - 原始内容
- `file` - 解析的配置
- `hash` - SHA-256 哈希

### ExecApprovalsResolved
解析的批准配置
- `path` - 文件路径
- `socketPath` - Socket 路径
- `token` - Socket 令牌
- `defaults` - 默认策略（必需）
- `agent` - 代理策略（必需）
- `allowlist` - 白名单数组
- `file` - 原始文件

### ExecAllowlistEntry
白名单条目
- `id` - 唯一 ID
- `pattern` - 命令模式
- `lastUsedAt` - 最后使用时间
- `lastUsedCommand` - 最后使用的命令
- `lastResolvedPath` - 最后解析的路径

## 主要常量

### 默认值
- `DEFAULT_SECURITY = "deny"`
- `DEFAULT_ASK = "on-miss"`
- `DEFAULT_ASK_FALLBACK = "deny"`
- `DEFAULT_AUTO_ALLOW_SKILLS = false`
- `DEFAULT_SOCKET = "~/.openclaw/exec-approvals.sock"`
- `DEFAULT_FILE = "~/.openclaw/exec-approvals.json"`

## 主要函数

### normalizeExecApprovals(file): ExecApprovalsFile
- **功能**: 规范化执行批准配置
- **参数**: 配置文件对象
- **返回值**: 规范化的配置
- **实现逻辑**:
  1. 合并遗留的 `default` 配置到 `main`
  2. 强制转换白名单为数组
  3. 为白名单条目添加 ID
  4. 返回规范化配置

### readExecApprovalsSnapshot(): ExecApprovalsSnapshot
- **功能**: 读取配置快照
- **返回值**: 快照对象
- **实现逻辑**:
  1. 检查文件是否存在
  2. 读取并解析 JSON
  3. 规范化配置
  4. 计算哈希
  5. 返回快照

### loadExecApprovals(): ExecApprovalsFile
- **功能**: 加载配置
- **返回值**: 配置文件对象
- **实现逻辑**:
  1. 文件不存在时返回默认配置
  2. 读取和解析 JSON
  3. 验证版本号
  4. 规范化配置

### saveExecApprovals(file): void
- **功能**: 保存配置
- **实现逻辑**:
  1. 确保目录存在
  2. 写入 JSON（2 空格缩进）
  3. 设置权限为 0600

### ensureExecApprovals(): ExecApprovalsFile
- **功能**: 确保配置存在并有效
- **返回值**: 更新后的配置
- **实现逻辑**:
  1. 加载现有配置
  2. 规范化
  3. 确保 Socket 配置完整
  4. 保存更新后的配置

### resolveExecApprovals(agentId?, overrides?): ExecApprovalsResolved
- **功能**: 解析执行批准配置
- **参数**:
  - `agentId` - 代理 ID
  - `overrides` - 策略覆盖
- **返回值**: 解析的配置对象
- **实现逻辑**:
  1. 确保配置存在
  2. 合并默认、通配符和代理特定策略
  3. 应用覆盖
  4. 解析白名单
  5. 返回完整配置

### resolveExecApprovalsFromFile(params): ...
- **功能**: 从文件解析执行批准配置
- **参数**: `{ file, agentId?, overrides?, path?, socketPath?, token? }`
- **返回值**: 解析的配置
- **实现逻辑**: 同 `resolveExecApprovals`，但支持自定义文件和路径

### requiresExecApproval(params): boolean
- **功能**: 判断是否需要批准
- **参数**:
  - `ask` - 询问策略
  - `security` - 安全策略
  - `analysisOk` - 分析是否通过
  - `allowlistSatisfied` - 白名单是否满足
- **返回值**: 是否需要批准
- **实现逻辑**:
  - `always` → 总是需要
  - `on-miss` + `allowlist` + 不满足 → 需要

### recordAllowlistUse(approvals, agentId, entry, command, resolvedPath?): void
- **功能**: 记录白名单使用
- **参数**: 配置、代理 ID、条目、命令、解析路径
- **实现逻辑**:
  1. 更新条目的使用时间戳
  2. 更新最后使用的命令和路径
  3. 保存配置

### addAllowlistEntry(approvals, agentId, pattern): void
- **功能**: 添加白名单条目
- **参数**: 配置、代理 ID、模式字符串
- **实现逻辑**:
  1. 避免重复
  2. 生成唯一 ID
  3. 添加到白名单
  4. 保存配置

### minSecurity(a, b): ExecSecurity
- **功能**: 返回更严格的安全策略
- **返回值**: 更严格的策略（deny < allowlist < full）

### maxAsk(a, b): ExecAsk
- **功能**: 返回更激进的询问策略
- **返回值**: 更激进的策略（off < on-miss < always）

### requestExecApprovalViaSocket(params): Promise<ExecApprovalDecision | null>
- **功能**: 通过 Socket 请求执行批准
- **参数**:
  - `socketPath` - Socket 路径
  - `token` - 认证令牌
  - `request` - 请求对象
  - `timeoutMs` - 超时时间
- **返回值**: 批准决策或 null
- **实现逻辑**:
  1. 创建 Unix Socket 客户端
  2. 连接并发送请求
  3. 接收响应
  4. 超时处理
  5. 返回决策

## 辅助函数

### hashExecApprovalsRaw(raw): string
- **功能**: 计算配置哈希
- **返回值**: SHA-256 十六进制

### expandHome(value): string
- **功能**: 展开主目录路径
- **返回值**: 绝对路径

### normalizeAllowlistPattern(value): string | null
- **功能**: 规范化白名单模式
- **返回值**: 小写模式或 null

### coerceAllowlistEntries(allowlist): ExecAllowlistEntry[] | undefined
- **功能**: 强制转换白名单为数组
- **返回值**: 标准化的数组

### ensureAllowlistIds(allowlist): ExecAllowlistEntry[] | undefined
- **功能**: 确保白名单条目有 ID
- **返回值**: 更新后的数组

### mergeLegacyAgent(current, legacy): ExecApprovalsAgent
- **功能**: 合并遗留代理配置
- **返回值**: 合并后的配置

### ensureDir(filePath): void
- **功能**: 确保目录存在

### generateToken(): string
- **功能**: 生成随机令牌
- **返回值**: base64url 编码的 24 字节随机数

### normalizeSecurity(value, fallback): ExecSecurity
- **功能**: 规范化安全策略
- **返回值**: 有效策略或回退值

### normalizeAsk(value, fallback): ExecAsk
- **功能**: 规范化询问策略
- **返回值**: 有效策略或回退值

## 主要依赖
- `node:crypto` - 随机和哈希
- `node:fs` - 文件系统
- `node:net` - Socket 连接
- `node:os` - 操作系统
- `node:path` - 路径处理
- `../routing/session-key.js` - 默认代理 ID
- `./exec-approvals-analysis.js` - 分析工具
- `./exec-approvals-allowlist.js` - 白名单工具

## 使用场景
- 命令执行批准
- 白名单管理
- 安全策略控制
- Socket 通信
- 配置持久化

## 代码行数
553 行

## 重要特性
- Unix Socket 通信
- 配置版本控制
- 白名单管理
- 遗留配置迁移
- 策略合并和覆盖
- 哈希验证

## 策略优先级

### 安全策略
1. 代理特定
2. 通配符 `*`
3. 默认
4. 覆盖

### 询问策略
1. 代理特定
2. 通配符 `*`
3. 默认
4. 覆盖

### 白名单
1. 代理特定
2. 通配符 `*`
（合并去重）

## Socket 通信

### 请求格式
```json
{
  "type": "request",
  "token": "xxx",
  "id": "uuid",
  "request": { ... }
}
```

### 响应格式
```json
{
  "type": "decision",
  "decision": "allow-once" | "allow-always" | "deny"
}
```

## 决策类型
- `"allow-once"` - 仅批准一次
- `"allow-always"` - 永久批准（添加到白名单）
- `"deny"` - 拒绝

## 使用示例

### 加载配置
```typescript
const approvals = loadExecApprovals();
// {
//   version: 1,
//   socket: { path: "~/.openclaw/exec-approvals.sock", token: "..." },
//   defaults: { security: "allowlist", ask: "on-miss" },
//   agents: {
//     "main": { security: "deny", allowlist: [...] }
//   }
// }
```

### 解析策略
```typescript
const resolved = resolveExecApprovals("myagent");
// {
//   path: "~/.openclaw/exec-approvals.json",
//   socketPath: "~/.openclaw/exec-approvals.sock",
//   token: "abc123",
//   defaults: { security: "allowlist", ask: "on-miss", ... },
//   agent: { security: "deny", ask: "always", ... },
//   allowlist: [...],
//   file: ...
// }
```

### 判断是否需要批准
```typescript
const needApproval = requiresExecApproval({
  ask: "on-miss",
  security: "allowlist",
  analysisOk: true,
  allowlistSatisfied: false
});
// true

const needApproval = requiresExecApproval({
  ask: "always",
  security: "deny",
  analysisOk: true,
  allowlistSatisfied: true
});
// true
```

### 通过 Socket 请求批准
```typescript
const decision = await requestExecApprovalViaSocket({
  socketPath: "~/.openclaw/exec-approvals.sock",
  token: "secret-token",
  request: {
    command: "ls -la",
    cwd: "/home/user",
    agentId: "myagent"
  },
  timeoutMs: 15000
});
// "allow-once" | "allow-always" | "deny" | null
```

### 添加白名单
```typescript
addAllowlistEntry(approvals, "myagent", "ls*");
// 添加模式并保存

addAllowlistEntry(approvals, "myagent", "npm run*");
// 添加另一个模式
```

### 记录使用
```typescript
recordAllowlistUse(approvals, "myagent", entry, "ls -la", "/usr/bin/ls");
// 更新条目的使用时间和命令
```

### 策略比较
```typescript
minSecurity("deny", "allowlist")
// "deny" (更严格)

minSecurity("full", "allowlist")
// "allowlist" (更严格)

maxAsk("off", "on-miss")
// "on-miss" (更激进)

maxAsk("always", "on-miss")
// "always" (更激进)
```

### 配置快照
```typescript
const snapshot = readExecApprovalsSnapshot();
// {
//   path: "~/.openclaw/exec-approvals.json",
//   exists: true,
//   raw: "{...}",
//   file: {...},
//   hash: "a1b2c3..."
// }
```

## 默认配置

### 最严格（默认）
```json
{
  "security": "deny",
  "ask": "on-miss",
  "autoAllowSkills": false
}
```

### 推荐配置
```json
{
  "security": "allowlist",
  "ask": "on-miss",
  "autoAllowSkills": true
}
```

### 宽松配置
```json
{
  "security": "full",
  "ask": "off",
  "autoAllowSkills": true
}
```

## 安全考虑

### 文件权限
- 配置文件权限：0600
- 仅所有者读写

### Socket 安全
- 使用 Unix Socket
- 令牌认证
- 超时保护

### 默认安全
- 默认拒绝执行
- 白名单模式更安全
- 不信任任何命令

## 白名单模式

### 模式匹配
- 精确匹配：`ls -la`
- 通配符：`ls*`, `npm run*`
- 大小写不敏感

### 使用记录
- 最后使用时间
- 最后使用的命令
- 最后解析的路径

## 遗留配置迁移

### default → main
```typescript
// 遗留
{
  "default": { "security": "deny" }
}

// 迁移后
{
  "main": { "security": "deny" }
}
```

### 白名单强制转换
```typescript
// 遗留（数组索引）
{
  "allowlist": ["ls -la", "cat*"]
}

// 迁移后
{
  "allowlist": [
    { "id": "uuid1", "pattern": "ls -la" },
    { "id": "uuid2", "pattern": "cat*" }
  ]
}
```
