# 文件解释：src/cli/windows-argv.ts

## 文件路径
`src/cli/windows-argv.ts`

## 文件用途
Windows 平台参数标准化工具，清理和规范化 Windows 命令行参数。

## 主要类/函数

### `normalizeWindowsArgv(argv: string[]): string[]`
标准化 Windows 参数数组，执行以下清理操作：

1. **控制字符过滤**：移除所有 ASCII 控制字符（32 和 127 之外的字符）
2. **引号剥离**：移除参数两端的单引号和双引号
3. **空格修剪**：移除多余空格
4. **UNC 路径清理**：移除 `\\\\?\` 前缀（Windows 长路径前缀）
5. **可执行路径去除**：移除重复的 Node.js 可执行路径参数

**检测可执行路径**：通过以下方式识别：
- 完整路径匹配 process.execPath
- 基础名称匹配
- 以 `node.exe` 结尾
- 包含 `node.exe` 且文件存在

## 辅助函数

#### `stripControlChars(value: string): string`
移除字符串中的控制字符（保留可打印字符）。

#### `normalizeArg(value: string): string`
标准化单个参数：移除控制字符、引号和空格。

#### `normalizeCandidate(value: string): string`
标准化候选参数，额外移除 UNC 路径前缀。

#### `isExecPath(value: string): boolean`
判断字符串是否为可执行路径，使用多种匹配策略。

## 主要依赖

- `node:fs` - 文件系统访问
- `node:path` - 路径处理

## 重要逻辑说明

1. **Windows 特定问题**：Windows 命令行传递参数时可能包含：
   - 控制字符
   - 额外的引号
   - UNC 长路径前缀（`\\\\?\`）
   - 重复的可执行路径

2. **清理策略**：
   - 仅在 Windows 平台（`win32`）执行
   - 保留前 3 个参数（通常足够）
   - 使用多种方式识别可执行路径以提高兼容性

3. **可执行路径识别**：使用宽松的匹配规则，确保正确识别各种形式的 Node.js 路径。

## 行数统计
79 行
