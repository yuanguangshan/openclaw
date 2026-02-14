# utils/reaction-level.ts 解读

**文件路径**: `src/utils/reaction-level.ts`

## 文件用途
表情反应级别解析和配置模块，用于控制消息在不同通道中的表情反应行为。

## 主要类型定义

### ReactionLevel
反应级别枚举：
- `off`: 关闭所有反应
- `ack`: 仅确认反应（如处理时显示 👀）
- `minimal`: 最小代理反应（稀疏使用）
- `extensive`: 广泛代理反应（自由使用）

### ResolvedReactionLevel
解析后的反应级别：
- `level`: 原始级别
- `ackEnabled`: ACK 反应是否启用
- `agentReactionsEnabled`: 代理反应是否启用
- `agentReactionGuidance`: 代理反应指导级别

## 主要常量

### LEVELS
有效的反应级别集合：`["off", "ack", "minimal", "extensive"]`

## 主要函数

### parseLevel(value: unknown): ParseResult
- **功能**: 解析反应级别
- **参数**: 任意类型的值
- **返回值**: 解析结果对象
- **返回类型**:
  - `{ kind: "missing" }` - 值缺失
  - `{ kind: "invalid" }` - 值无效
  - `{ kind: "ok"; value: ReactionLevel }` - 解析成功

### resolveReactionLevel(params): ResolvedReactionLevel
- **功能**: 解析并解析反应级别配置
- **参数**:
  - `value`: 配置值
  - `defaultLevel`: 默认级别
  - `invalidFallback`: 无效时的回退值
- **返回值**: 解析后的反应级别配置
- **实现逻辑**:
  1. 解析配置值
  2. 根据解析结果选择有效级别
  3. 映射到具体的行为标志

## 级别行为映射

### off
- `ackEnabled`: false
- `agentReactionsEnabled`: false
- `agentReactionGuidance`: undefined

### ack
- `ackEnabled`: true
- `agentReactionsEnabled`: false
- `agentReactionGuidance`: undefined

### minimal
- `ackEnabled`: false
- `agentReactionsEnabled`: true
- `agentReactionGuidance`: "minimal"

### extensive
- `ackEnabled`: false
- `agentReactionsEnabled`: true
- `agentReactionGuidance`: "extensive"

## 使用场景
- 消息处理反馈
- 用户交互体验
- 通道特定行为
- 噪音级别控制

## 代码行数
75 行

## 重要特性
- 清晰的级别定义
- 灵活的默认值处理
- 详细的解析结果
- 行为标志分离

## 反应级别对比

| 级别 | ACK 反应 | 代理反应 | 指导 | 使用场景 |
|------|----------|----------|------|----------|
| `off` | ✗ | ✗ | - | 无反馈 |
| `ack` | ✓ | ✗ | - | 基本确认 |
| `minimal` | ✗ | ✓ | 稀疏 | 适度反馈 |
| `extensive` | ✗ | ✓ | 自由 | 丰富反馈 |

## 使用示例
```typescript
// 解析 "minimal" 级别
const config = resolveReactionLevel({
  value: "minimal",
  defaultLevel: "off",
  invalidFallback: "ack"
});
// {
//   level: "minimal",
//   ackEnabled: false,
//   agentReactionsEnabled: true,
//   agentReactionGuidance: "minimal"
// }

// 处理缺失值
const config2 = resolveReactionLevel({
  value: undefined,
  defaultLevel: "ack",
  invalidFallback: "minimal"
});
// {
//   level: "ack",
//   ackEnabled: true,
//   agentReactionsEnabled: false,
//   agentReactionGuidance: undefined
// }

// 处理无效值
const config3 = resolveReactionLevel({
  value: "invalid",
  defaultLevel: "off",
  invalidFallback: "minimal"
});
// {
//   level: "minimal",
//   ackEnabled: false,
//   agentReactionsEnabled: true,
//   agentReactionGuidance: "minimal"
// }
```