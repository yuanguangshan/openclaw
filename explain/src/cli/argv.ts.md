# 文件解释：src/cli/argv.ts

## 文件路径
`src/cli/argv.ts`

## 文件用途
CLI 参数解析工具集，提供一系列辅助函数来解析和处理命令行参数。

## 主要类/函数

### 帮助和版本标志检测

#### `hasHelpOrVersion(argv: string[]): boolean`
检查参数数组中是否包含帮助或版本标志（`-h`, `--help`, `-v`, `-V`, `--version`）。

### 标志解析

#### `hasFlag(argv: string[], name: string): boolean`
检查参数数组中是否包含指定的标志（忽略 `--` 后的参数）。

#### `getFlagValue(argv: string[], name: string): string | null | undefined`
获取指定标志的值。支持两种格式：
- `--flag value`
- `--flag=value`

返回 `undefined` 表示标志不存在，`null` 表示标志存在但无值。

#### `getVerboseFlag(argv: string[], options?: { includeDebug?: boolean }): boolean`
检查是否启用详细输出模式，包括 `--verbose` 和可选的 `--debug` 标志。

#### `getPositiveIntFlagValue(argv: string[], name: string): number | null | undefined`
获取指定标志的正整数值。

### 命令路径解析

#### `getCommandPath(argv: string[], depth?: number): string[]`
从参数数组中提取命令路径（非标志参数），可以指定深度。

#### `getPrimaryCommand(argv: string[]): string | null`
获取主命令（第一个非标志参数）。

### 参数构建

#### `buildParseArgv(params): string[]`
构建规范化的参数数组，确保包含 node 和程序名称，用于 Commander.js 解析。

### 迁移判断

#### `shouldMigrateStateFromPath(path: string[]): boolean`
根据命令路径判断是否需要迁移状态。对于只读命令（如 health、status、config get 等）返回 false。

#### `shouldMigrateState(argv: string[]): boolean`
基于完整的 argv 判断是否需要迁移状态。

## 辅助函数

#### `isValueToken(arg: string | undefined): boolean`
判断参数是否为值（非标志、非 `--` 终止符、或者是负数）。

#### `parsePositiveInt(value: string): number | undefined`
解析正整数，无效时返回 `undefined`。

#### `isNodeExecutable(executable: string): boolean`
判断字符串是否为 Node.js 可执行文件名。

#### `isBunExecutable(executable: string): boolean`
判断字符串是否为 Bun 可执行文件名。

## 主要依赖

无外部依赖，纯工具函数。

## 重要逻辑说明

1. **标志终止符**：所有标志解析函数都会在遇到 `--` 时停止处理，这是 POSIX 标准约定。

2. **负数处理**：`isValueToken` 函数特别处理以 `-` 开头但后面跟随数字的情况（如 `-1.5`），将其视为值而非标志。

3. **命令识别**：命令路径解析会跳过所有以 `-` 开头的参数，只收集实际的命令和子命令。

4. **迁移优化**：`shouldMigrateState` 函数避免在只读操作中执行昂贵的状态迁移，提升性能。

## 行数统计
176 行
