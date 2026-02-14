# channels/registry.ts 解读

**文件路径**: `src/channels/registry.ts`

## 文件用途
通道注册表模块，定义所有支持的消息通道类型、元数据和规范化函数，提供通道的集中管理。

## 主要类型定义

### ChatChannelId
聊天通道标识符类型
- `"telegram"` - Telegram Bot
- `"whatsapp"` - WhatsApp Web
- `"discord"` - Discord Bot
- `"irc"` - IRC
- `"googlechat"` - Google Chat
- `"slack"` - Slack Bot
- `"signal"` - Signal
- `"imessage"` - iMessage

### ChannelMeta
通道元数据（通用类型）
- `id: string` - 通道 ID
- `label: string` - 显示标签
- `selectionLabel?: string` - 选择界面标签
- `detailLabel?: string` - 详细标签
- `docsPath: string` - 文档路径
- `docsLabel?: string` - 文档标签
- `blurb: string` - 简短描述
- `systemImage?: string` - 系统图标名称
- `selectionDocsPrefix?: string` - 文档前缀
- `selectionDocsOmitLabel?: boolean` - 是否省略文档标签
- `selectionExtras?: string[]` - 额外信息

## 主要常量

### CHAT_CHANNEL_ORDER
聊天通道顺序数组（固定顺序）
```
["telegram", "whatsapp", "discord", "irc", "googlechat", "slack", "signal", "imessage"]
```

### DEFAULT_CHAT_CHANNEL
默认聊天通道：`"whatsapp"`

### CHAT_CHANNEL_META
所有通道的元数据映射表

### CHAT_CHANNEL_ALIASES
通道别名映射
```typescript
{
  imsg: "imessage",
  "internet-relay-chat": "irc",
  "google-chat": "googlechat",
  gchat: "googlechat",
}
```

## 主要函数

### listChatChannels(): ChatChannelMeta[]
- **功能**: 获取所有聊天通道的元数据列表
- **参数**: 无
- **返回值**: 通道元数据数组（按 CHAT_CHANNEL_ORDER 顺序）
- **实现逻辑**: 映射 CHAT_CHANNEL_ORDER 中的每个 ID 到元数据

### listChatChannelAliases(): string[]
- **功能**: 获取所有通道别名列表
- **参数**: 无
- **返回值**: 别名字符串数组
- **实现逻辑**: 返回 CHAT_CHANNEL_ALIASES 的键

### getChatChannelMeta(id: ChatChannelId): ChatChannelMeta
- **功能**: 获取指定通道的元数据
- **参数**: 通道 ID
- **返回值**: 通道元数据对象
- **实现逻辑**: 从 CHAT_CHANNEL_META 查找并返回

### normalizeChatChannelId(raw?: string | null): ChatChannelId | null
- **功能**: 规范化聊天通道 ID
- **参数**: 原始通道 ID（字符串）
- **返回值**: 规范化的通道 ID 或 null（无效）
- **实现逻辑**:
  1. 转小写并去除空白
  2. 解析别名
  3. 验证是否在支持的通道列表中
  4. 返回规范化结果

### normalizeChannelId(raw?: string | null): ChatChannelId | null
- **功能**: 规范化通道 ID（别名函数）
- **参数**: 原始通道 ID
- **返回值**: 规范化的通道 ID 或 null
- **实现逻辑**: 委托给 `normalizeChatChannelId`

### normalizeAnyChannelId(raw?: string | null): ChannelId | null
- **功能**: 规范化任意通道 ID（包括插件通道）
- **参数**: 原始通道 ID
- **返回值**: 规范化的通道 ID 或 null
- **实现逻辑**:
  1. 转小写并去除空白
  2. 查询插件注册表
  3. 匹配通道 ID 或别名
  4. 返回匹配的通道 ID

### formatChannelPrimerLine(meta: ChatChannelMeta): string
- **功能**: 格式化通道简介行
- **参数**: 通道元数据
- **返回值**: 格式化的简介字符串
- **实现逻辑**: `{label}: {blurb}`

### formatChannelSelectionLine(meta, docsLink): string
- **功能**: 格式化通道选择界面行
- **参数**:
  - `meta` - 通道元数据
  - `docsLink` - 文档链接生成函数
- **返回值**: 格式化的选择行
- **实现逻辑**:
  1. 构建文档链接
  2. 添加额外信息
  3. 组合成完整的格式化字符串

## 主要依赖
- `./plugins/types.js` - 通道类型定义
- `../plugins/runtime.js` - 插件运行时

## 使用场景
- 通道选择界面
- 配置验证和规范化
- 文档链接生成
- 通道信息展示
- 插件系统集成

## 代码行数
192 行

## 重要特性
- 集中的通道管理
- 灵活的别名支持
- 顺序固定的通道列表
- 丰富的元数据支持
- 轻量级插件集成
- 大小写不敏感的规范化

## 支持的通道

### Telegram
- **ID**: `telegram`
- **别名**: 无
- **类型**: Bot API
- **描述**: 最简单的入门方式 — 使用 @BotFather 注册机器人
- **图标**: paperplane

### WhatsApp
- **ID**: `whatsapp`
- **别名**: 无
- **类型**: QR 链接
- **默认**: 是
- **描述**: 使用自己的号码；建议使用独立手机 + eSIM
- **图标**: message

### Discord
- **ID**: `discord`
- **别名**: 无
- **类型**: Bot API
- **描述**: 当前支持非常好
- **图标**: bubble.left.and.bubble.right

### IRC
- **ID**: `irc`
- **别名**: `"internet-relay-chat"`
- **类型**: Server + Nick
- **描述**: 经典 IRC 网络，支持 DM/频道路由和配对控制
- **图标**: network

### Google Chat
- **ID**: `googlechat`
- **别名**: `"google-chat"`, `gchat`
- **类型**: Chat API
- **描述**: Google Workspace Chat 应用，使用 HTTP webhook
- **图标**: message.badge

### Slack
- **ID**: `slack`
- **别名**: 无
- **类型**: Socket Mode
- **描述**: 支持（Socket Mode）
- **图标**: number

### Signal
- **ID**: `signal`
- **别名**: 无
- **类型**: signal-cli
- **描述**: signal-cli 链接设备；更多设置（David Reagans: "Hop on Discord"）
- **图标**: antenna.radiowaves.left.and.right

### iMessage
- **ID**: `imessage`
- **别名**: `imsg`
- **类型**: imsg
- **描述**: 仍在开发中
- **图标**: message.fill

## 规范化规则

### 大小写
- 所有输入转为小写
- 返回值始终是小写的规范 ID

### 空白处理
- 去除前后空白
- 空字符串返回 null

### 别名解析
- 优先使用别名表
- 别名不存在则使用原始值
- 不在支持列表中返回 null

## 使用示例

### 列出所有通道
```typescript
const channels = listChatChannels();
// [
//   { id: "telegram", label: "Telegram", ... },
//   { id: "whatsapp", label: "WhatsApp", ... },
//   ...
// ]
```

### 规范化通道 ID
```typescript
normalizeChatChannelId("WHATSAPP");      // "whatsapp"
normalizeChatChannelId("gchat");         // "googlechat"
normalizeChatChannelId("imsg");          // "imessage"
normalizeChatChannelId("unknown");       // null
normalizeChatChannelId("");              // null
```

### 获取通道元数据
```typescript
const meta = getChatChannelMeta("whatsapp");
// {
//   id: "whatsapp",
//   label: "WhatsApp",
//   selectionLabel: "WhatsApp (QR link)",
//   detailLabel: "WhatsApp Web",
//   docsPath: "/channels/whatsapp",
//   docsLabel: "whatsapp",
//   blurb: "works with your own number; recommend a separate phone + eSIM.",
//   systemImage: "message",
// }
```

### 格式化输出
```typescript
// 简介行
const primer = formatChannelPrimerLine(meta);
// "WhatsApp: works with your own number; recommend a separate phone + eSIM."

// 选择行
const selection = formatChannelSelectionLine(meta, (path, label) => {
  return `[${label || path}](https://docs.openclaw.ai${path})`;
});
// "WhatsApp — works with your own number... Docs: [whatsapp](https://docs.openclaw.ai/channels/whatsapp)"
```

## 插件集成

### 添加新通道
1. 在 `CHAT_CHANNEL_ORDER` 中添加 ID
2. 在 `CHAT_CHANNEL_META` 中添加元数据
3. 在插件注册表中注册插件
4. 保持协议 ID 同步

### 插件通道
```typescript
// 规范化包括插件在内的所有通道
const id = normalizeAnyChannelId("msteams");
// 返回 "msteams"（如果插件已注册）
// 返回 null（如果插件未注册）
```

## 设计考虑

### 为什么使用固定顺序
- UI 显示一致性
- 测试可预测性
- 文档顺序统一

### 为什么提供别名
- 用户友好性（`gchat` vs `googlechat`）
- 向后兼容性（`imsg` vs `imessage`）
- 灵活的命名

### 为什么区分 `normalizeChatChannelId` 和 `normalizeAnyChannelId`
- **`normalizeChatChannelId`**: 仅核心通道，轻量级（不加载插件）
- **`normalizeAnyChannelId`**: 所有通道（包括插件），需要插件注册表

## 元数据字段说明

### 文档相关
- `docsPath`: 文档相对路径
- `docsLabel`: 文档链接文本
- `selectionDocsPrefix`: 文档前缀文本
- `selectionDocsOmitLabel`: 是否省略标签（仅显示链接）

### UI 相关
- `label`: 主显示标签
- `selectionLabel`: 选择界面标签
- `detailLabel`: 详细视图标签
- `systemImage`: 系统图标名称（SF Symbols）
- `blurb`: 简短描述
- `selectionExtras`: 额外信息（如网站链接）

## 扩展性

### 添加新通道
```typescript
// 1. 添加到顺序数组
CHAT_CHANNEL_ORDER.push("newchannel");

// 2. 添加元数据
CHAT_CHANNEL_META.newchannel = {
  id: "newchannel",
  label: "New Channel",
  // ...
};

// 3. 可选：添加别名
CHAT_CHANNEL_ALIASES["new"] = "newchannel";
```

### 添加插件通道
- 在插件入口点注册
- 提供正确的元数据和别名
- 保持协议 ID 与核心通道一致
