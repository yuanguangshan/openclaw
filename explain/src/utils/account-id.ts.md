# 文件解释：src/utils/account-id.ts

## 文件路径
`src/utils/account-id.ts`

## 文件用途
账号 ID 标准化工具。

## 主要类/函数

### `normalizeAccountId(value?: string): string | undefined`
标准化账号 ID：
- 如果不是字符串，返回 `undefined`
- 去除首尾空格
- 空字符串返回 `undefined`
- 有效字符串返回去除空格后的值

## 主要依赖

无外部依赖。

## 重要逻辑说明

这是一个简单的规范化工具，确保账号 ID 格式一致。

## 行数统计
8 行
