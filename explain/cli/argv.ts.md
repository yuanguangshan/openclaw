# cli/argv.ts 解读

**文件路径**: `src/cli/argv.ts`

## 文件用途
命令行参数解析工具模块，提供灵活的命令行参数处理和分析功能。

## 常量定义

### HELP_FLAGS
帮助标志集合: `["-h", "--help"]`

### VERSION_FLAGS
版本标志集合: `["-v", "-V", "--version"]`

### FLAG_TERMINATOR
标志终止符: `"--"`

## 主要函数

### hasHelpOrVersion(argv: string[]): boolean
- **功能**: 检查参数数组中是否包含帮助或版本标志
- **参数**: 命令行参数数组
- **返回值**: 布尔值

### isValueToken(arg: string | undefined): boolean
- **功能**: 判断参数是否为值标记（非标志）
- **参数**: 命令行参数
- **返回值**: 布尔值
- **实现逻辑**:
  - 空值或终止符 → false
  - 以 `-` 开头但为数字 → true
  - 不以 `-` 开头 → true

### parsePositiveInt(value: string): number | undefined
- **功能**: 解析正整数
- **参数**: 字符串值
- **返回值**: 解析后的数字或 undefined
- **验证**: 必须为正整数

### hasFlag(argv: string[], name: string): boolean
- **功能**: 检查参数中是否包含指定标志
- **参数**:
  - `argv`: 命令行参数数组
  - `name`: 标志名称
- **返回值**: 布尔值
- **实现逻辑**: 遍历参数，遇到终止符停止

### getFlagValue(argv: string[], name: string): string | null | undefined
- **功能**: 获取标志的值
- **参数**:
  - `argv`: 命令行参数数组
  - `name`: 标志名称
- **返回值**: 标志值（null表示存在但无值，undefined表示不存在）
- **支持格式**:
  - `--flag value`
  - `--flag=value`

### getVerboseFlag(argv: string[], options?: { includeDebug?: boolean }): boolean
- **功能**: 获取详细输出标志状态
- **参数**:
  - `argv`: 命令行参数
  - `options`: 选项（是否包含 --debug）
- **返回值**: 布尔值
- **检查标志**: `--verbose`, `--debug`（可选）

### getPositiveIntFlagValue(argv: string[], name: string): number | null | undefined
- **功能**: 获取正整数标志值
- **参数**:
  - `argv`: 命令行参数
  - `name`: 标志名称
- **返回值**: 解析后的正整数
- **验证**: 使用 `parsePositiveInt`

### getCommandPath(argv: string[], depth = 2): string[]
- **功能**: 获取命令路径（命令和子命令）
- **参数**:
  - `argv`: 命令行参数
  - `depth`: 深度（默认2）
- **返回值**: 命令路径数组
- **实现逻辑**:
  - 跳过标志
  - 收集非标志参数
  - 遇到终止符停止

### getPrimaryCommand(argv: string[]): string | null
- **功能**: 获取主命令
- **参数**: 命令行参数
- **返回值**: 主命令字符串或 null
- **实现**: 获取命令路径的第一个元素

### buildParseArgv(params): string[]
- **功能**: 构建和解析命令行参数
- **参数**:
  - `programName`: 程序名称
  - `rawArgs`: 原始参数
  - `fallbackArgv`: 备用参数
- **返回值**: 标准化的参数数组
- **实现逻辑**:
  - 处理程序名称
  - 识别 Node.js 和 Bun 执行器
  - 标准化参数格式

### isNodeExecutable(executable: string): boolean
- **功能**: 判断是否为 Node.js 执行器
- **参数**: 可执行文件名
- **返回值**: 布尔值

### isBunExecutable(executable: string): boolean
- **功能**: 判断是否为 Bun 执行器
- **参数**: 可执行文件名
- **返回值**: 布尔值

### shouldMigrateStateFromPath(path: string[]): boolean
- **功能**: 判断给定命令路径是否应该迁移状态
- **参数**: 命令路径
- **返回值**: 布尔值
- **排除命令**: health, status, sessions, config get/unset, models list/status, memory status, agent

### shouldMigrateState(argv: string[]): boolean
- **功能**: 判断当前命令是否应该迁移状态
- **参数**: 命令行参数
- **返回值**: 布尔值

## 使用场景
- 命令行参数解析
- CLI 标志处理
- 命令路径分析
- 状态迁移决策

## 代码行数
176 行

## 重要特性
- 支持多种参数格式
- 智能执行器检测
- 类型安全的数值解析
- 灵活的标志处理

## 使用示例
```typescript
// 解析命令行参数
const argv = ["node", "openclaw", "message", "send", "--to", "user@example.com", "--verbose"];

// 检查标志
const isVerbose = hasFlag(argv, "--verbose"); // true

// 获取标志值
const to = getFlagValue(argv, "--to"); // "user@example.com"

// 获取命令路径
const commands = getCommandPath(argv); // ["message", "send"]

// 获取主命令
const primary = getPrimaryCommand(argv); // "message"
```