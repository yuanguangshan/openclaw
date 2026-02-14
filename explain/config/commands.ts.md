# config/commands.ts 解读

**文件路径**: `src/config/commands.ts`

## 文件用途
原生命令和技能配置解析模块，负责根据配置和通道类型决定是否启用原生命令功能。

## 主要函数

### resolveAutoDefault(providerId?: ChannelId): boolean
- **功能**: 根据通道类型解析自动默认值
- **参数**: 通道 ID
- **返回值**: 布尔值
- **实现逻辑**:
  - Discord → true
  - Telegram → true
  - Slack → false
  - 其他 → false

### resolveNativeSkillsEnabled(params): boolean
- **功能**: 解析是否启用原生技能
- **参数**:
  - `providerId`: 通道 ID
  - `providerSetting`: 通道级设置
  - `globalSetting`: 全局设置
- **返回值**: 布尔值
- **实现逻辑**:
  1. 通道级设置优先于全局设置
  2. true → 启用
  3. false → 禁用
  4. undefined/auto → 使用自动默认值

### resolveNativeCommandsEnabled(params): boolean
- **功能**: 解析是否启用原生命令
- **参数**:
  - `providerId`: 通道 ID
  - `providerSetting`: 通道级设置
  - `globalSetting`: 全局设置
- **返回值**: 布尔值
- **实现逻辑**:
  1. 通道级设置优先于全局设置
  2. true → 启用
  3. false → 禁用
  4. undefined/auto → 使用启发式判断

### isNativeCommandsExplicitlyDisabled(params): boolean
- **功能**: 检查原生命令是否被显式禁用
- **参数**:
  - `providerSetting`: 通道级设置
  - `globalSetting`: 全局设置
- **返回值**: 布尔值
- **实现逻辑**:
  - 通道级设置为 false → true
  - 通道级未设置，全局设置为 false → true
  - 其他情况 → false

## 主要依赖
- `../channels/plugins/types.js` - 通道 ID 类型
- `./types.js` - 配置类型定义
- `../channels/plugins/index.js` - 通道 ID 标准化

## 使用场景
- 通道命令功能控制
- 技能启用/禁用决策
- 命令冲突处理
- 用户体验优化

## 代码行数
65 行

## 重要特性
- 灵活的配置层级
- 智能默认值
- 明确的优先级规则
- 禁用状态检测

## 配置行为
| 设置值 | 效果 |
|--------|------|
| true | 强制启用 |
| false | 强制禁用 |
| undefined/auto | 使用通道类型默认值 |

## 通道默认行为
| 通道类型 | 默认启用 |
|----------|----------|
| Discord | ✓ |
| Telegram | ✓ |
| Slack | ✗ |
| 其他 | ✗ |

## 使用示例
```typescript
// Discord - 默认启用
const discordEnabled = resolveNativeCommandsEnabled({
  providerId: "discord",
  providerSetting: undefined,
  globalSetting: undefined
});
// true

// Slack - 默认禁用，但全局设置为启用
const slackEnabled = resolveNativeCommandsEnabled({
  providerId: "slack",
  providerSetting: undefined,
  globalSetting: true
});
// true

// Telegram - 通道级禁用覆盖全局设置
const telegramEnabled = resolveNativeCommandsEnabled({
  providerId: "telegram",
  providerSetting: false,
  globalSetting: true
});
// false

// 检查是否显式禁用
const isExplicitlyDisabled = isNativeCommandsExplicitlyDisabled({
  providerSetting: false,
  globalSetting: undefined
});
// true
```