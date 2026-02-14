# 文件解释：src/utils/boolean.ts

## 文件路径
`src/utils/boolean.ts`

## 文件用途
布尔值解析工具，将各种格式的字符串解析为布尔值。

## 主要类/函数

### `DEFAULT_TRUTHY`
默认的真值集合：`["true", "1", "yes", "on"]`。

### `DEFAULT_FALSY`
默认的假值集合：`["false", "0", "no", "off"]`。

### `parseBooleanValue(value: unknown, options?: BooleanParseOptions): boolean | undefined`
将值解析为布尔值。

**逻辑**：
- 如果是布尔值，直接返回
- 如果不是字符串，返回 `undefined`
- 空字符串返回 `undefined`
- 匹配真值集合返回 `true`
- 匹配假值集合返回 `false`
- 其他情况返回 `undefined`

**BooleanParseOptions**：
- `truthy?: string[]` - 自定义真值集合
- `falsy?: string[]` - 自定义假值集合

## 类型定义

```typescript
type BooleanParseOptions = {
  truthy?: string[];
  falsy?: string[];
};
```

## 主要依赖

无外部依赖。

## 重要逻辑说明

1. **大小写不敏感**：解析时将字符串转为小写进行比较。

2. **Set 优化**：使用 Set 存储默认值集合以提高查找效率，只在需要自定义时创建新的 Set。

3. **严格类型**：只有明确的真值和假值才返回对应的布尔值，其他情况返回 `undefined`，避免强制转换带来的问题。

## 行数统计
37 行
