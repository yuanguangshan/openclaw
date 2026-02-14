# utils/safe-json.ts 解读

**文件路径**: `src/utils/safe-json.ts`

## 文件用途
安全的 JSON 序列化工具，处理特殊类型（如 BigInt、函数、错误对象等），避免序列化失败。

## 主要函数

### safeJsonStringify(value: unknown): string | null
- **功能**: 安全地将任意值序列化为 JSON 字符串
- **参数**: 任意类型的值
- **返回值**: JSON 字符串或 null（序列化失败时）
- **实现逻辑**:
  1. 使用 `JSON.stringify` 和替换函数
  2. 处理特殊类型：
     - `BigInt`: 转换为字符串
     - `Function`: 替换为 "[Function]"
     - `Error`: 提取 name、message 和 stack
     - `Uint8Array`: 转换为 base64 编码的字符串
  3. 捕获异常，失败时返回 null

## 特殊类型处理

### BigInt
- **问题**: JSON 不支持 BigInt
- **解决方案**: 转换为字符串表示
- **示例**: `123n` → `"123"`

### Function
- **问题**: 函数无法序列化
- **解决方案**: 替换为占位符 "[Function]"
- **示例**: `() => {}` → `"[Function]"`

### Error
- **问题**: 错误对象包含不可枚举属性
- **解决方案**: 提取可用的错误信息
- **结果格式**:
  ```json
  {
    "name": "Error",
    "message": "Error message",
    "stack": "Error: Error message\n    at ..."
  }
  ```

### Uint8Array
- **问题**: 二进制数据无法直接序列化
- **解决方案**: 转换为 base64 编码
- **结果格式**:
  ```json
  {
    "type": "Uint8Array",
    "data": "base64-encoded-string"
  }
  ```

## 使用场景
- 日志记录
- 调试输出
- 错误报告
- 数据持久化
- 网络传输

## 代码行数
22 行

## 重要特性
- 类型安全
- 异常安全
- 特殊类型支持
- 零依赖

## 使用示例
```typescript
// 序列化 BigInt
const bigintValue = safeJsonStringify({ id: 123456789012345678901234567890n });
// '{"id":"123456789012345678901234567890"}'

// 序列化函数
const funcValue = safeJsonStringify({ handler: () => console.log("hello") });
// '{"handler":"[Function]"}'

// 序列化错误
const errorValue = safeJsonStringify({
  error: new Error("Something went wrong")
});
// '{"error":{"name":"Error","message":"Something went wrong","stack":"..."}}'

// 序列化二进制数据
const binaryValue = safeJsonStringify({
  data: new Uint8Array([72, 101, 108, 108, 111])
});
// '{"data":{"type":"Uint8Array","data":"SGVsbG8="}}'

// 处理循环引用（会失败，返回 null）
const obj: any = {};
obj.self = obj;
const circularValue = safeJsonStringify(obj);
// null
```