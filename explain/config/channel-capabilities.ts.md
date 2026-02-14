# config/channel-capabilities.ts 解读

**文件路径**: `src/config/channel-capabilities.ts`

## 文件用途
通道能力配置解析模块，负责从 OpenClaw 配置中提取和规范化通道特定的能力设置。

## 主要类型定义

### CapabilitiesConfig
能力配置类型，基于 TelegramCapabilitiesConfig

## 主要函数

### isStringArray(value: unknown): value is string[]
- **功能**: 类型保护函数，判断值是否为字符串数组
- **参数**: 任意类型的值
- **返回值**: 布尔值（类型守卫）
- **实现逻辑**: 检查是否为数组且所有元素都是字符串

### normalizeCapabilities(capabilities: CapabilitiesConfig | undefined): string[] | undefined
- **功能**: 标准化能力配置
- **参数**: 能力配置（可能为对象格式或数组格式）
- **返回值**: 标准化后的字符串数组或 undefined
- **实现逻辑**:
  1. 仅处理字符串数组格式
  2. 优雅处理对象格式（如 `{ inlineButtons: "dm" }`）
  3. 去除空白字符
  4. 过滤空字符串
  5. 返回非空数组或 undefined

### resolveAccountCapabilities(params): string[] | undefined
- **功能**: 解析账户能力配置
- **参数**:
  - `cfg`: 配置对象（包含 accounts 和 capabilities）
  - `accountId`: 账户 ID
- **返回值**: 能力字符串数组或 undefined
- **实现逻辑**:
  1. 尝试精确匹配账户 ID
  2. 如果没有精确匹配，尝试不区分大小写的匹配
  3. 账户级配置优先于全局配置
  4. 返回标准化后的能力列表

### resolveChannelCapabilities(params): string[] | undefined
- **功能**: 解析通道能力配置
- **参数**:
  - `cfg`: OpenClaw 配置（部分）
  - `channel`: 通道 ID
  - `accountId`: 账户 ID
- **返回值**: 能力字符串数组或 undefined
- **实现逻辑**:
  1. 标准化通道 ID
  2. 查找通道特定配置
  3. 委托给 `resolveAccountCapabilities` 解析账户级能力
  4. 支持多种配置路径（channels 对象或顶级配置）

## 主要依赖
- `./config.js` - OpenClaw 配置类型
- `./types.telegram.js` - Telegram 能力配置类型
- `../channels/plugins/index.js` - 通道 ID 标准化
- `../routing/session-key.js` - 账户 ID 标准化

## 使用场景
- 通道功能特性控制
- 账户权限管理
- 功能开关配置
- 通道特定行为定制

## 代码行数
74 行

## 重要特性
- 多层级配置支持（全局、通道、账户）
- 大小写不敏感的账户匹配
- 优雅的格式处理
- 配置优先级正确

## 配置层级优先级
1. 账户级配置（最高优先级）
2. 通道级配置
3. 全局配置（最低优先级）

## 使用示例
```typescript
const config = {
  channels: {
    telegram: {
      capabilities: ["inlineButtons", "reactions"],
      accounts: {
        "bot1": {
          capabilities: ["inlineButtons", "threads"]
        }
      }
    }
  }
};

// 解析全局能力
const globalCaps = resolveChannelCapabilities({
  cfg: config,
  channel: "telegram"
});
// ["inlineButtons", "reactions"]

// 解析账户特定能力
const accountCaps = resolveChannelCapabilities({
  cfg: config,
  channel: "telegram",
  accountId: "bot1"
});
// ["inlineButtons", "threads"]
```