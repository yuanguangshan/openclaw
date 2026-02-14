# 文件解释：src/shared/config-eval.ts

## 文件路径
`src/shared/config-eval.ts`

## 文件用途
配置评估和运行时辅助工具。

## 主要类/函数

### `isTruthy(value: unknown): boolean`
检查值是否为真值：
- `null` 或 `undefined` 返回 `false`
- 布尔值直接返回
- 数字：非零为 `true`
- 字符串：非空为 `true`
- 其他类型返回 `true`

### `resolveConfigPath(config: unknown, pathStr: string): unknown`
从配置对象中按点号路径解析值。例如：`"agents.defaults.model"` 会解析 `config.agents.defaults.model`。

### `isConfigPathTruthyWithDefaults(config: unknown, pathStr: string, defaults: Record<string, boolean>): boolean`
解析配置路径并判断是否为真值，如果路径不存在且有默认值则使用默认值。

### `resolveRuntimePlatform(): string`
返回当前运行平台（`process.platform`）。

### `hasBinary(bin: string): boolean`
检查系统 PATH 中是否存在指定的可执行文件。支持 Windows 的可执行文件扩展名（.EXE, .CMD, .BAT, .COM）。

## 辅助函数

#### `windowsPathExtensions(): string[]`
返回 Windows 平台的可执行文件扩展名列表。

## 主要依赖

- `node:fs` - 文件系统访问
- `node:path` - 路径处理

## 重要逻辑说明

1. **路径解析**：`resolveConfigPath` 使用点号分隔符来访问嵌套对象属性，类似 JavaScript 的属性访问语法。

2. **真值判断**：`isTruthy` 遵循 JavaScript 的弱类型转换规则，但更严格（空字符串为 false）。

3. **二进制检测**：`hasBinary` 遍历 PATH 中的每个目录，尝试查找可执行文件。在 Windows 上会自动尝试常见扩展名。

4. **默认值支持**：`isConfigPathTruthyWithDefaults` 允许为配置路径提供默认值，当路径不存在时回退到默认值。

## 行数统计
72 行
