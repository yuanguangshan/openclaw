# 文件解释：src/cli/cli-name.ts

## 文件路径
`src/cli/cli-name.ts`

## 文件用途
CLI 名称解析和替换工具，用于识别和标准化 CLI 命令中的程序名称。

## 主要类/函数

### `DEFAULT_CLI_NAME`
默认 CLI 名称常量，值为 "openclaw"。

### `resolveCliName(argv?: string[]): string`
从参数数组中解析 CLI 名称。如果参数中包含已知 CLI 名称，返回该名称；否则返回默认名称 "openclaw"。

### `replaceCliName(command: string, cliName?: string): string`
替换命令字符串中的 CLI 名称前缀。使用正则表达式匹配常见的 CLI 命令格式（如 `pnpm openclaw`、`npx openclaw` 等），并将其替换为指定的 CLI 名称。

## 主要依赖

- `node:path` - 路径处理

## 重要逻辑说明

1. **CLI 名称识别**：维护已知 CLI 名称的集合（目前只有 "openclaw"），通过检查参数路径的基础名称来识别。

2. **命令前缀替换**：使用正则表达式 `/^(?:((?:pnpm|npm|bunx|npx)\s+))?(openclaw)\b/` 匹配命令字符串，保留包管理器前缀（如 pnpm、npm 等），只替换实际的 CLI 名称。

3. **默认行为**：当无法识别时，默认使用 "openclaw" 作为 CLI 名称。

## 行数统计
31 行
