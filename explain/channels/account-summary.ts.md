# channels/account-summary.ts 解读

**文件路径**: `src/channels/account-summary.ts`

## 文件用途
通道账户摘要工具，用于构建和格式化通道账户的快照信息和允许列表。

## 主要函数

### buildChannelAccountSnapshot(params): ChannelAccountSnapshot
- **功能**: 构建通道账户快照
- **参数**:
  - `plugin`: 通道插件
  - `account`: 账户对象
  - `cfg`: OpenClaw 配置
  - `accountId`: 账户 ID
  - `enabled`: 是否启用
  - `configured`: 是否已配置
- **返回值**: 通道账户快照
- **实现逻辑**:
  1. 调用插件的 `describeAccount` 方法获取账户描述
  2. 合并基础信息和插件描述
  3. 返回完整的快照对象

### formatChannelAllowFrom(params): string[]
- **功能**: 格式化通道允许来源列表
- **参数**:
  - `plugin`: 通道插件
  - `cfg`: OpenClaw 配置
  - `accountId`: 账户 ID（可选）
  - `allowFrom`: 允许来源列表
- **返回值**: 格式化后的允许来源数组
- **实现逻辑**:
  1. 优先使用插件的 `formatAllowFrom` 方法
  2. 回退到默认格式化：字符串转换、去空白、过滤空值

## 主要依赖
- `../config/config.js` - OpenClaw 配置类型
- `./plugins/types.core.js` - 核心通道类型
- `./plugins/types.plugin.js` - 通道插件类型

## 使用场景
- 账户状态显示
- 配置信息汇总
- 允许列表格式化
- UI 信息展示

## 代码行数
37 行

## 重要特性
- 插件化支持
- 灵活的描述格式
- 统一的快照结构
- 可扩展的格式化

## 使用示例
```typescript
// 构建账户快照
const snapshot = buildChannelAccountSnapshot({
  plugin: telegramPlugin,
  account: { username: "mybot", token: "***" },
  cfg: openclawConfig,
  accountId: "bot1",
  enabled: true,
  configured: true
});
// {
//   enabled: true,
//   configured: true,
//   username: "mybot",
//   accountId: "bot1",
//   // ... 其他插件特定信息
// }

// 格式化允许列表
const allowFrom = formatChannelAllowFrom({
  plugin: discordPlugin,
  cfg: openclawConfig,
  accountId: "guild1",
  allowFrom: [123456789012345678, "user_id_123"]
});
// ["123456789012345678", "user_id_123"]
```