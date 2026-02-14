# utils/normalize-secret-input.ts 解读

**文件路径**: `src/utils/normalize-secret-input.ts`

## 文件用途
密钥/令牌输入标准化工具，处理用户复制粘贴凭证时常见的格式问题，特别是换行符问题。

## 主要函数

### normalizeSecretInput(value: unknown): string
- **功能**: 标准化密钥/令牌输入
- **参数**: 任意类型的输入值
- **返回值**: 标准化后的字符串（非字符串返回空字符串）
- **实现逻辑**:
  1. 移除所有换行符（包括 `\r\n`、`\u2028`、`\u2029`）
  2. 去除首尾空白字符
  3. 保留字符串内部的普通空格（避免破坏 `"Bearer <token>"` 格式）

### normalizeOptionalSecretInput(value: unknown): string | undefined
- **功能**: 标准化可选的密钥输入
- **参数**: 任意类型的输入值
- **返回值**: 标准化后的字符串或 undefined（空字符串返回 undefined）
- **实现逻辑**: 调用 `normalizeSecretInput`，空字符串转换为 undefined

## 设计原理

### 问题背景
用户从网页或其他来源复制 API 密钥/令牌时，经常会意外包含：
- 换行符（`\r\n`）
- 行分隔符（`\u2028`、`\u2029`）
- 首尾空白字符

### 解决方案
- **移除所有换行符**: 确保密钥连续性
- **保留内部空格**: 避免破坏特定格式的令牌（如 Bearer tokens）
- **去除首尾空白**: 标准化输入格式

## 使用场景
- API 密钥输入处理
- 访问令牌验证
- 凭证表单处理
- 配置文件密钥读取

## 代码行数
21 行

## 重要特性
- 安全处理（不修改内部空格）
- 跨平台换行符支持
- 类型安全
- 可选值支持

## 使用示例
```typescript
// 处理包含换行的 API 密钥
const apiKey = "sk-1234567890abcdef\r\n";
const normalized = normalizeSecretInput(apiKey);
// "sk-1234567890abcdef"

// 处理 Bearer token
const token = "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
const normalized = normalizeSecretInput(token);
// "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

// 处理可选密钥
const optionalKey = null;
const normalized = normalizeOptionalSecretInput(optionalKey);
// undefined

// 处理空字符串
const emptyKey = "  \r\n  ";
const normalized = normalizeSecretInput(emptyKey);
// ""
```