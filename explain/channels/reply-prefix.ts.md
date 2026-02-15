# channels/reply-prefix.ts 解读

**文件路径**: `src/channels/reply-prefix.ts`

## 文件用途
回复前缀上下文模块，提供代理回复的模型相关上下文配置。

## 主要类型定义

### ModelSelectionContext
模型选择上下文（来自 GetReplyOptions 的 onModelSelected）

### ReplyPrefixContextBundle
回复前缀上下文包
- `prefixContext` - 前缀上下文
- `responsePrefix` - 回复前缀
- `responsePrefixContextProvider` - 上下文提供器函数
- `onModelSelected` - 模型选择回调

### ReplyPrefixOptions
回复前缀选项（从 Bundle 中选择的属性）

## 主要函数

### createReplyPrefixContext(params): ReplyPrefixContextBundle
- **功能**: 创建回复前缀上下文
- **参数**: `{ cfg, agentId, channel?, accountId? }`
- **返回值**: 上下文包
- **实现逻辑**:
  1. 解析代理身份名称
  2. 创建前缀上下文（identityName）
  3. 创建模型选择回调
  4. 解析有效消息配置的回复前缀
  5. 返回完整的上下文包

### createReplyPrefixOptions(params): ReplyPrefixOptions
- **功能**: 创建回复前缀选项
- **参数**: `{ cfg, agentId, channel?, accountId? }`
- **返回值**: 选项对象
- **实现逻辑**: 创建上下文包并返回选项属性

## 主要依赖
- `../auto-reply/types.js` - GetReplyOptions 类型
- `../config/config.js` - 配置类型
- `../agents/identity.js` - 代理身份工具
- `../auto-reply/reply/response-prefix-template.js` - 前缀模板

## 使用场景
- 代理回复配置
- 模型选择
- 消息前缀设置

## 代码行数
63 行

## 重要特性
- 代理身份集成
- 模型选择回调
- 上下文提供器模式

## 前缀上下文结构
```typescript
{
  identityName: "My Agent",
  provider: "anthropic",
  model: "claude-opus-4-6",
  modelFull: "anthropic/claude-opus-4-6",
  thinkingLevel: "medium"
}
```

## 模型选择回调
模型选择时更新：
- provider
- model（短名称）
- modelFull（完整名称）
- thinkingLevel
