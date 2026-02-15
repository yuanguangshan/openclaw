# channels/plugins/types.plugin.ts 解读

**文件路径**: `src/channels/plugins/types.plugin.ts`

## 文件用途
通道插件核心类型定义，定义通道插件的标准接口和契约。

## 主要类型定义

### ChannelConfigUiHint
配置 UI 提示
- `label` - 标签
- `help` - 帮助文本
- `advanced` - 高级选项
- `sensitive` - 敏感信息
- `placeholder` - 占位符
- `itemTemplate` - 项目模板

### ChannelConfigSchema
配置模式
- `schema` - Zod 模式对象
- `uiHints` - UI 提示映射

### ChannelPlugin<ResolvedAccount, Probe, Audit>
通道插件类型
- `id` - 通道 ID
- `meta` - 通道元数据
- `capabilities` - 通道能力
- `defaults` - 默认配置
  - `queue` - 队列默认值
- `reload` - 重载配置
- `onboarding` - CLI 入门适配器
- `config` - 配置适配器
- `configSchema` - 配置模式
- `setup` - 设置适配器
- `pairing` - 配对适配器
- `security` - 安全适配器
- `groups` - 群组适配器
- `mentions` - 提及适配器
- `outbound` - 出站适配器
- `status` - 状态适配器
- `gatewayMethods` - 网关方法列表
- `gateway` - 网关适配器
- `auth` - 认证适配器
- `elevated` - 提升适配器
- `commands` - 命令适配器
- `streaming` - 流式传输适配器
- `threading` - 线程适配器
- `messaging` - 消息传递适配器
- `agentPrompt` - 代理提示适配器
- `directory` - 目录适配器
- `resolver` - 解析器适配器
- `actions` - 消息操作适配器
- `heartbeat` - 心跳适配器
- `agentTools` - 代理工具（通道拥有）

## 主要依赖
- `./onboarding-types.js` - 入门类型
- `./types.adapters.js` - 适配器类型
- `./types.core.js` - 核心类型

## 使用场景
- 通道插件开发
- 插件注册
- 配置验证
- 适配器实现

## 代码行数
85 行

## 重要特性
- 泛型支持（ResolvedAccount, Probe, Audit）
- 全面的适配器接口
- 配置模式验证
- CLI 入门支持
