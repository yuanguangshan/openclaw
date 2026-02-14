# utils/usage-format.ts 解读

**文件路径**: `src/utils/usage-format.ts`

## 文件用途
使用量格式化和成本估算工具模块，提供 Token 计数格式化、成本计算和模型价格配置解析功能。

## 主要类型定义

### ModelCostConfig
模型成本配置：
- `input`: 输入 Token 价格（每百万）
- `output`: 输出 Token 价格（每百万）
- `cacheRead`: 缓存读取价格（每百万）
- `cacheWrite`: 缓存写入价格（每百万）

### UsageTotals
使用量总计：
- `input`: 输入 Token 数量
- `output`: 输出 Token 数量
- `cacheRead`: 缓存读取 Token 数量
- `cacheWrite`: 缓存写入 Token 数量
- `total`: 总计 Token 数量

## 主要函数

### formatTokenCount(value?: number): string
- **功能**: 格式化 Token 计数
- **参数**: Token 数量
- **返回值**: 格式化后的字符串
- **格式规则**:
  - ≥ 1,000,000: 显示为百万（如 "1.5m"）
  - ≥ 10,000: 显示为千（如 "15k"）
  - ≥ 1,000: 显示为千（如 "1.5k"）
  - < 1,000: 显示整数

### formatUsd(value?: number): string | undefined
- **功能**: 格式化美元金额
- **参数**: 金额数值
- **返回值**: 格式化后的字符串或 undefined
- **格式规则**:
  - ≥ $1.00: `$1.23`
  - ≥ $0.01: `$0.01`
  - < $0.01: `$0.0045`（4位小数）

### resolveModelCostConfig(params): ModelCostConfig | undefined
- **功能**: 解析模型成本配置
- **参数**:
  - `provider`: 提供商标识符
  - `model`: 模型标识符
  - `config`: OpenClaw 配置
- **返回值**: 模型成本配置或 undefined
- **实现逻辑**:
  1. 从配置中查找提供商和模型
  2. 返回匹配的模型成本配置

### estimateUsageCost(params): number | undefined
- **功能**: 估算使用成本
- **参数**:
  - `usage`: 使用量数据
  - `cost`: 模型成本配置
- **返回值**: 估算的成本（美元）或 undefined
- **计算公式**:
  ```
  成本 = (输入 × 输入价格 +
         输出 × 输出价格 +
         缓存读取 × 缓存读取价格 +
         缓存写入 × 缓存写入价格) / 1,000,000
  ```

## 辅助函数

### toNumber(value: number | undefined): number
- **功能**: 安全转换为数字
- **参数**: 数字值
- **返回值**: 数字或 0

## 使用场景
- 成本计算和显示
- 使用量统计
- 价格配置管理
- 账单分析

## 代码行数
87 行

## 重要特性
- 灵活的格式化
- 成本估算
- 配置解析
- 类型安全

## 格式化示例

### Token 计数
```typescript
formatTokenCount(500);
// "500"

formatTokenCount(1500);
// "1.5k"

formatTokenCount(15000);
// "15k"

formatTokenCount(1500000);
// "1.5m"
```

### 美元金额
```typescript
formatUsd(1.5);
// "$1.50"

formatUsd(0.05);
// "$0.05"

formatUsd(0.005);
// "$0.0050"
```

### 成本估算
```typescript
const cost = estimateUsageCost({
  usage: {
    input: 1000,
    output: 500,
    cacheRead: 200,
    cacheWrite: 100
  },
  cost: {
    input: 5,      // $5 per million
    output: 15,    // $15 per million
    cacheRead: 2,  // $2 per million
    cacheWrite: 4   // $4 per million
  }
});
// (1000×5 + 500×15 + 200×2 + 100×4) / 1,000,000
// = (5000 + 7500 + 400 + 400) / 1,000,000
// = 13,300 / 1,000,000
// = 0.0133 (美元)
```