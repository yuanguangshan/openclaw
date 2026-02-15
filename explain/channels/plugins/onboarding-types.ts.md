# channels/plugins/onboarding-types.ts 解读

**文件路径**: `src/channels/plugins/onboarding-types.ts`

## 文件用途
通道入门类型定义，提供通道配置、账户 ID 提示和状态管理的类型接口。

## 主要类型定义

### SetupChannelsOptions
通道设置选项
- `allowDisable` - 允许禁用
- `allowSignalInstall` - 允许安装 Signal
- `onSelection` - 选择回调
- `accountIds` - 账户 ID 映射
- `onAccountId` - 账户回调
- `promptAccountIds` - 提示账户 ID
- `whatsappAccountId` - WhatsApp 账户 ID
- `promptWhatsAppAccountId` - 提示 WhatsApp 账户 ID
- `onWhatsAppAccountId` - WhatsApp 账户回调
- `forceAllowFromChannels` - 强制允许的通道
- `skipStatusNote` - 跳过状态说明
- `skipDmPolicyPrompt` - 跳过 DM 策略提示
- `skipConfirm` - 跳过确认
- `quickstartDefaults` - 快速开始默认值
- `initialSelection` - 初始选择

### PromptAccountIdParams
提示账户 ID 参数
- `cfg` - 配置
- `prompter` - 提示器
- `label` - 标签
- `currentId` - 当前 ID
- `listAccountIds` - 列出账户 ID 的函数
- `defaultAccountId` - 默认账户 ID

### PromptAccountId = (params) => Promise<string>
提示账户 ID 函数类型

### ChannelOnboardingStatus
通道入门状态
- `channel` - 通道 ID
- `configured` - 是否已配置
- `statusLines` - 状态行数组
- `selectionHint` - 选择提示
- `quickstartScore` - 快速开始分数

### ChannelOnboardingStatusContext
通道入门状态上下文
- `cfg` - 配置
- `options` - 设置选项
- `accountOverrides` - 账户覆盖

### ChannelOnboardingConfigureContext
通道入门配置上下文
- `cfg` - 配置
- `runtime` - 运行时环境
- `prompter` - 提示器
- `options` - 设置选项
- `accountOverrides` - 账户覆盖
- `shouldPromptAccountIds` - 是否提示账户 ID
- `forceAllowFrom` - 是否强制允许

### ChannelOnboardingResult
通道入门结果
- `cfg` - 配置
- `accountId` - 账户 ID

### ChannelOnboardingDmPolicy
通道 DM 策略
- `label` - 标签
- `channel` - 通道 ID
- `policyKey` - 策略键
- `allowFromKey` - 允许列表键
- `getCurrent` - 获取当前策略
- `setPolicy` - 设置策略
- `promptAllowFrom` - 提示允许列表

### ChannelOnboardingAdapter
通道入门适配器
- `channel` - 通道 ID
- `getStatus` - 获取状态
- `configure` - 配置
- `dmPolicy` - DM 策略

## 主要依赖
- `../../config/config.js` - 配置类型
- `../../config/types.js` - DM 策略类型
- `../../runtime.js` - 运行时类型
- `../../wizard/prompts.js` - 提示器类型
- `./types.js` - 通道 ID 类型

## 使用场景
- 通道入门流程
- 账户 ID 提示
- 状态管理和显示
- 配置向导
- DM 策略配置

## 代码行数
87 行

## 重要特性
- 类型驱动的配置流程
- 灵活的选项控制
- 回调支持
- 状态追踪
