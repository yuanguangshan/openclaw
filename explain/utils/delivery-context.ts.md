# utils/delivery-context.ts 解读

**文件路径**: `src/utils/delivery-context.ts`

## 文件用途
管理消息投递上下文的工具模块，用于处理消息投递的目标信息，包括通道、接收者、账户 ID 和线程 ID 等。

## 主要类型定义

### DeliveryContext
消息投递上下文接口：
- `channel`: 通道名称
- `to`: 目标接收者
- `accountId`: 账户 ID
- `threadId`: 线程 ID

### DeliveryContextSessionSource
会话投递上下文来源接口，包含当前和历史投递信息。

## 主要函数

### normalizeDeliveryContext(context?: DeliveryContext): DeliveryContext | undefined
- **功能**: 标准化投递上下文
- **参数**: 原始投递上下文
- **返回值**: 标准化后的投递上下文
- **实现逻辑**:
  - 标准化通道名称
  - 去除空白字符
  - 处理线程 ID 的数字和字符串类型
  - 过滤无效值

### normalizeSessionDeliveryFields(source?: DeliveryContextSessionSource)
- **功能**: 标准化会话投递字段
- **返回值**: 包含标准化后的投递上下文和最后字段的对象
- **实现逻辑**: 合并当前和历史投递信息

### deliveryContextFromSession(entry)
- **功能**: 从会话条目提取投递上下文
- **参数**: 会话条目
- **返回值**: 提取的投递上下文
- **实现逻辑**: 从多个可能的位置收集投递信息

### mergeDeliveryContext(primary?, fallback?)
- **功能**: 合并两个投递上下文
- **参数**:
  - `primary`: 主要投递上下文
  - `fallback`: 备用投递上下文
- **返回值**: 合并后的投递上下文
- **实现逻辑**: 使用主要上下文，缺少字段时使用备用上下文

### deliveryContextKey(context?)
- **功能**: 生成投递上下文的唯一键
- **参数**: 投递上下文
- **返回值**: 格式化的键字符串
- **实现逻辑**: 格式为 `channel|to|accountId|threadId`

## 主要依赖
- `./account-id.js` - 账户 ID 标准化
- `./message-channel.js` - 消息通道标准化

## 使用场景
- 消息投递目标管理
- 会话状态维护
- 多通道消息路由
- 线程化对话管理

## 代码行数
141 行

## 重要特性
- 类型安全，使用 TypeScript 接口
- 完善的标准化处理
- 支持多种数据源合并
- 生成唯一键用于缓存和查找