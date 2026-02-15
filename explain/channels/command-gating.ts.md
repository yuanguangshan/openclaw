# channels/command-gating.ts 解读

**文件路径**: `src/channels/command-gating.ts`

## 文件用途
命令门控模块，控制命令是否被授权以及是否应该阻止。

## 主要类型定义

### CommandAuthorizer
命令授权者
- `configured` - 是否已配置
- `allowed` - 是否允许

### CommandGatingModeWhenAccessGroupsOff
访问组关闭时的门控模式
- `"allow"` - 允许所有
- `"deny"` - 拒绝所有
- `"configured"` - 遵循配置

## 主要函数

### resolveCommandAuthorizedFromAuthorizers(params): boolean
- **功能**: 解析命令是否被授权
- **参数**:
  - `useAccessGroups` - 是否使用访问组
  - `authorizers` - 授权者列表
  - `modeWhenAccessGroupsOff` - 访问组关闭时的模式
- **返回值**: 是否授权
- **实现逻辑**:
  1. 不使用访问组时检查模式
  2. 检查任何已配置的授权者
  3. 检查所有允许的授权者
  4. 默认返回配置且允许

### resolveControlCommandGate(params): { commandAuthorized: boolean; shouldBlock: boolean }
- **功能**: 解析控制命令门控
- **参数**:
  - `useAccessGroups` - 是否使用访问组
  - `authorizers` - 授权者列表
  - `allowTextCommands` - 是否允许文本命令
  - `hasControlCommand` - 是否有控制命令
  - `modeWhenAccessGroupsOff` - 访问组关闭时的模式
- **返回值**: `{ commandAuthorized, shouldBlock }`
- **实现逻辑**:
  1. 检查命令授权
  2. 检查是否应该阻止（文本命令 + 未授权 + 有控制命令）
  3. 返回决策

## 主要依赖
- 无外部依赖（纯类型和逻辑）

## 使用场景
- 命令访问控制
- 访问组管理
- 安全门控

## 代码行数
46 行

## 重要特性
- 多层授权检查
- 访问组支持
- 文本命令特殊处理

## 授权决策

### resolveCommandAuthorizedFromAuthorizers
```
不使用访问组:
  - mode=allow → true
  - mode=deny → false
  - 任何 configured → true
  - 所有 allowed → true
  - configured + allowed → true
  - configured + !allowed → false
  - 默认: true

使用访问组:
  - 遵循授权者配置
```

### resolveControlCommandGate
```
commandAuthorized: 从授权者解析

shouldBlock:
  - allowTextCommands && hasControlCommand && !commandAuthorized
  - 否则 false
```

## 使用示例

### 授权检查
```typescript
const authorized = resolveCommandAuthorizedFromAuthorizers({
  useAccessGroups: false,
  authorizers: [
    { configured: false, allowed: true }
  ],
  modeWhenAccessGroupsOff: "allow"
});
// true
```

### 控制命令门控
```typescript
const gate = resolveControlCommandGate({
  useAccessGroups: false,
  authorizers: [
    { configured: true, allowed: false }
  ],
  allowTextCommands: true,
  hasControlCommand: true,
  modeWhenAccessGroupsOff: "deny"
});
// { commandAuthorized: false, shouldBlock: true }
```

### 访问组模式
```typescript
// 允许所有
resolveCommandAuthorizedFromAuthorizers({
  useAccessGroups: false,
  authorizers: [],
  modeWhenAccessGroupsOff: "allow"
});
// true

// 拒绝所有
resolveCommandAuthorizedFromAuthorizers({
  useAccessGroups: false,
  authorizers: [],
  modeWhenAccessGroupsOff: "deny"
});
// false

// 遵循配置
resolveCommandAuthorizedFromAuthorizers({
  useAccessGroups: true,
  authorizers: [
    { configured: true, allowed: true }
  ]
});
// true
```
