# utils/queue-helpers.ts 解读

**文件路径**: `src/utils/queue-helpers.ts`

## 文件用途
队列管理和处理工具模块，提供队列溢出策略、去抖动、摘要生成等功能，用于管理消息、任务等队列数据。

## 主要类型定义

### QueueSummaryState
队列摘要状态：
- `dropPolicy`: 丢弃策略（"summarize" | "old" | "new"）
- `droppedCount`: 丢弃的项目计数
- `summaryLines`: 摘要行数组

### QueueDropPolicy
队列丢弃策略类型

### QueueState<T>
队列状态（泛型）：
- 继承自 QueueSummaryState
- `items`: 队列项目数组
- `cap`: 队列容量上限

## 主要函数

### elideQueueText(text: string, limit = 140): string
- **功能**: 截断过长的文本，添加省略号
- **参数**:
  - `text`: 原始文本
  - `limit`: 长度限制（默认 140）
- **返回值**: 截断后的文本
- **实现**: 超出限制时截断并添加 "…"

### buildQueueSummaryLine(text: string, limit = 160): string
- **功能**: 构建队列摘要行
- **参数**:
  - `text`: 原始文本
  - `limit`: 长度限制（默认 160）
- **返回值**: 摘要行
- **实现逻辑**:
  1. 压缩空白字符
  2. 截断过长文本

### shouldSkipQueueItem<T>(params): boolean
- **功能**: 判断是否应跳过队列项目（去重）
- **参数**:
  - `item`: 待检查的项目
  - `items`: 现有项目数组
  - `dedupe`: 去重函数（可选）
- **返回值**: 布尔值

### applyQueueDropPolicy<T>(params): boolean
- **功能**: 应用队列丢弃策略
- **参数**:
  - `queue`: 队列状态
  - `summarize`: 项目摘要函数
  - `summaryLimit`: 摘要限制（可选）
- **返回值**: 是否应该继续处理
- **丢弃策略**:
  - `new`: 丢弃新项目，不处理当前项目
  - `old`: 丢弃旧项目，处理当前项目
  - `summarize`: 丢弃旧项目并生成摘要

### waitForQueueDebounce(queue): Promise<void>
- **功能**: 等待队列防抖延迟
- **参数**:
  - `queue`: 包含 debounceMs 和 lastEnqueuedAt 的队列对象
- **返回值**: Promise
- **实现**: 等待指定时间后再次检查 lastEnqueuedAt

### buildQueueSummaryPrompt(params): string | undefined
- **功能**: 构建队列摘要提示
- **参数**:
  - `state`: 队列摘要状态
  - `noun`: 项目名词（单数形式）
  - `title`: 自定义标题（可选）
- **返回值**: 格式化的摘要字符串
- **实现逻辑**:
  1. 仅在策略为 "summarize" 且有丢弃项目时生成
  2. 包含标题和摘要行
  3. 重置摘要状态

### buildCollectPrompt<T>(params): string
- **功能**: 构建收集提示
- **参数**:
  - `title`: 标题
  - `items`: 项目数组
  - `summary`: 摘要（可选）
  - `renderItem`: 项目渲染函数
- **返回值**: 格式化的提示字符串

### hasCrossChannelItems<T>(items, resolveKey): boolean
- **功能**: 检查项目是否跨通道
- **参数**:
  - `items`: 项目数组
  - `resolveKey`: 项目键解析函数
- **返回值**: 布尔值
- **实现逻辑**:
  1. 检查是否有明确的跨通道标记
  2. 检查是否有多个不同的通道键
  3. 检查是否有未键化的项目

## 使用场景
- 消息队列管理
- 任务队列溢出处理
- 防抖动处理
- 队列摘要生成
- 多通道消息检测

## 代码行数
152 行

## 重要特性
- 灵活的丢弃策略
- 防抖动支持
- 摘要生成
- 跨通道检测
- 泛型支持

## 丢弃策略对比

| 策略 | 行为 | 适用场景 |
|------|------|----------|
| `new` | 丢弃新项目 | 保持最新消息 |
| `old` | 丢弃旧项目 | 保持最新内容 |
| `summarize` | 摘要旧项目 | 保留信息但减少体积 |

## 使用示例
```typescript
// 应用摘要策略
const queue: QueueState<string> = {
  items: ["msg1", "msg2", "msg3"],
  cap: 2,
  dropPolicy: "summarize",
  droppedCount: 0,
  summaryLines: []
};

const shouldProcess = applyQueueDropPolicy({
  queue,
  summarize: (item) => item,
  summaryLimit: 3
});
// queue.items 现在只保留最后2个，丢弃的生成摘要

// 防抖动等待
await waitForQueueDebounce({
  debounceMs: 1000,
  lastEnqueuedAt: Date.now()
});

// 检查跨通道
const isCrossChannel = hasCrossChannelItems(
  messages,
  (msg) => ({ key: msg.channel, cross: msg.crossChannel })
);

// 构建摘要提示
const summary = buildQueueSummaryPrompt({
  state: queue,
  noun: "message",
  title: "Some messages were dropped"
});
```