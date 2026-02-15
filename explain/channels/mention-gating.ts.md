# channels/mention-gating.ts 解读

**文件路径**: `src/channels/mention-gating.ts`

## 文件用途
提及门控模块，控制消息是否需要提及以及是否应该跳过。

## 主要类型定义

### MentionGateParams
提及门控参数
- `requireMention` - 是否需要提及
- `canDetectMention` - 是否能检测提及
- `wasMentioned` - 是否被提及
- `implicitMention` - 是否隐式提及
- `shouldBypassMention` - 是否绕过提及

### MentionGateResult
提及门控结果
- `effectiveWasMentioned` - 有效的提及状态
- `shouldSkip` - 是否应该跳过

### MentionGateWithBypassParams
带绕过的提及门控参数
- `isGroup` - 是否为群组
- `requireMention` - 是否需要提及
- `canDetectMention` - 是否能检测提及
- `wasMentioned` - 是否被提及
- `implicitMention` - 是否隐式提及
- `hasAnyMention` - 是否有任何提及
- `allowTextCommands` - 是否允许文本命令
- `hasControlCommand` - 是否有控制命令
- `commandAuthorized` - 命令是否授权

### MentionGateWithBypassResult
带绕过的提及门控结果
- 继承 MentionGateResult
- `shouldBypassMention` - 是否应该绕过提及

## 主要函数

### resolveMentionGating(params): MentionGateResult
- **功能**: 解析提及门控
- **参数**: MentionGateParams
- **返回值**: 门控结果
- **实现逻辑**:
  1. 确定有效的提及状态
  2. 检查是否需要提及
  3. 检查能否检测提及
  4. 返回是否应该跳过

### resolveMentionGatingWithBypass(params): MentionGateWithBypassResult
- **功能**: 解析提及门控（带绕过）
- **参数**: MentionGateWithBypassParams
- **返回值**: 门控结果
- **实现逻辑**:
  1. 调用基础门控
 2. 检查是否应该绕过（群组 + 未提及 + 文本命令）
  3. 返回结果

## 主要依赖
- 无外部依赖（纯类型和逻辑）

## 使用场景
- 群组消息过滤
- 提及检测
- 命令授权
- 消息路由

## 代码行数
60 行

## 重要特性
- 绕过逻辑支持
- 群组特殊处理
- 隐式提及支持

## 决策逻辑

### 基础门控
```
requireMention && canDetectMention && !effectiveWasMentioned → skip
```

### 带绕过的门控
```
isGroup && requireMention && !wasMentioned
&& !hasAnyMention && allowTextCommands
&& commandAuthorized → bypass
```

## 使用示例

### 基础门控
```typescript
const result = resolveMentionGating({
  requireMention: true,
  canDetectMention: true,
  wasMentioned: false,
  implicitMention: false
});
// { effectiveWasMentioned: false, shouldSkip: true }
```

### 带绕过的门控
```typescript
const result = resolveMentionGatingWithBypass({
  isGroup: true,
  requireMention: true,
  canDetectMention: true,
  wasMentioned: false,
  implicitMention: false,
  hasAnyMention: false,
  allowTextCommands: true,
  hasControlCommand: true,
  commandAuthorized: true
});
// { effectiveWasMentioned: false, shouldSkip: true, shouldBypassMention: true }
```

### 隐式提及
```typescript
const result = resolveMentionGating({
  requireMention: true,
  canDetectMention: true,
  wasMentioned: false,
  implicitMention: true  // 隐式提及
});
// { effectiveWasMentioned: true, shouldSkip: false }
```
