# utils/transcript-tools.ts 解读

**文件路径**: `src/utils/transcript-tools.ts`

## 文件用途
消息转录工具模块，从消息对象中提取和处理工具调用（tool call）信息，用于分析和统计 AI 代理的工具使用情况。

## 主要类型定义

### ToolResultCounts
工具结果统计：
- `total`: 工具结果总数
- `errors`: 错误数量

## 主要常量

### TOOL_CALL_TYPES
工具调用类型集合：`["tool_use", "toolcall", "tool_call"]`

### TOOL_RESULT_TYPES
工具结果类型集合：`["tool_result", "tool_result_error"]`

## 主要函数

### normalizeType(value: unknown): string
- **功能**: 标准化类型名称
- **参数**: 类型值
- **返回值**: 标准化后的字符串（非字符串返回空字符串）
- **实现**: 去除空白并转小写

### extractToolCallNames(message: Record<string, unknown>): string[]
- **功能**: 从消息中提取所有工具调用名称
- **参数**: 消息对象
- **返回值**: 工具名称数组（去重）
- **实现逻辑**:
  1. 检查消息级别的 `toolName` 或 `tool_name`
  2. 检查内容数组中的工具调用块
  3. 收集所有工具名称并去重

### hasToolCall(message: Record<string, unknown>): boolean
- **功能**: 判断消息是否包含工具调用
- **参数**: 消息对象
- **返回值**: 布尔值
- **实现**: 调用 `extractToolCallNames` 并检查结果

### countToolResults(message: Record<string, unknown>): ToolResultCounts
- **功能**: 统计消息中的工具结果数量
- **参数**: 消息对象
- **返回值**: 工具结果统计对象
- **实现逻辑**:
  1. 遍历内容数组
  2. 识别工具结果块
  3. 统计总数和错误数（检查 `is_error` 标志）

## 消息格式支持

### 消息级别工具名称
```json
{
  "toolName": "browser_navigate",
  "content": "..."
}
```

### 内容块格式
```json
{
  "content": [
    {
      "type": "tool_use",
      "name": "browser_navigate",
      "input": {...}
    },
    {
      "type": "tool_result",
      "content": "success",
      "is_error": false
    }
  ]
}
```

## 使用场景
- 工具调用分析
- 消息内容提取
- 使用统计
- 调试和日志

## 代码行数
74 行

## 重要特性
- 支持多种格式
- 去重处理
- 错误统计
- 类型安全

## 使用示例

### 提取工具调用名称
```typescript
const message = {
  content: [
    { type: "tool_use", name: "browser_navigate" },
    { type: "tool_use", name: "bash_exec" },
    { type: "tool_use", name: "browser_navigate" }
  ]
};

const names = extractToolCallNames(message);
// ["browser_navigate", "bash_exec"]
```

### 检查工具调用
```typescript
const hasCall = hasToolCall(message);
// true
```

### 统计工具结果
```typescript
const resultMessage = {
  content: [
    { type: "tool_result", content: "success" },
    { type: "tool_result", content: "error", is_error: true },
    { type: "tool_result", content: "success" }
  ]
};

const counts = countToolResults(resultMessage);
// { total: 3, errors: 1 }
```