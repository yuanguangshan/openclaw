# shared/config-eval.ts 解读

**文件路径**: `src/shared/config-eval.ts`

## 文件用途
配置评估和解析工具模块，提供配置值访问、真值判断和运行时平台检测功能。

## 主要函数

### isTruthy(value: unknown): boolean
- **功能**: 判断值是否为真值
- **参数**: 任意类型的值
- **返回值**: 布尔值
- **实现逻辑**:
  - undefined/null → false
  - boolean → 原值
  - number → 非 0 为 true
  - string → 非空字符串为 true
  - 其他类型 → true

### resolveConfigPath(config: unknown, pathStr: string): unknown
- **功能**: 通过点号路径访问配置对象
- **参数**:
  - `config`: 配置对象
  - `pathStr`: 点号分隔的路径（如 "server.port"）
- **返回值**: 找到的值或 undefined
- **实现逻辑**: 递归访问对象属性

### isConfigPathTruthyWithDefaults(config: unknown, pathStr: string, defaults: Record<string, boolean>): boolean
- **功能**: 判断配置路径值是否为真，支持默认值
- **参数**:
  - `config`: 配置对象
  - `pathStr`: 配置路径
  - `defaults`: 默认值映射
- **返回值**: 布尔值
- **实现逻辑**: 值不存在时使用默认值

### resolveRuntimePlatform(): string
- **功能**: 解析运行时平台
- **返回值**: 平台标识符（如 "darwin"、"win32"、"linux"）
- **实现**: 返回 `process.platform`

### hasBinary(bin: string): boolean
- **功能**: 检查系统 PATH 中是否存在指定二进制文件
- **参数**: `bin` - 二进制文件名
- **返回值**: 布尔值
- **实现逻辑**:
  - 解析 PATH 环境变量
  - Windows 平台处理扩展名（.EXE、.CMD、.BAT、.COM）
  - 使用 `fs.accessSync` 检查可执行权限
  - 遍历所有 PATH 目录查找

### windowsPathExtensions(): string[]
- **功能**: 获取 Windows 平台的可执行文件扩展名列表
- **返回值**: 扩展名数组（包括空字符串）
- **实现逻辑**: 从 PATHEXT 环境变量读取或使用默认值

## 主要依赖
- `node:fs` - 文件系统访问
- `node:path` - 路径处理
- Node.js `process` - 环境变量和平台信息

## 使用场景
- 配置文件解析和访问
- 环境检测和兼容性处理
- 二进制文件存在性检查
- 条件配置评估

## 代码行数
72 行

## 重要特性
- 类型安全的路径访问
- 跨平台二进制文件检测
- 灵活的真值判断逻辑
- 支持配置默认值

## 使用示例
```typescript
const config = { server: { port: 8080, enabled: true } };

// 访问嵌套配置
const port = resolveConfigPath(config, "server.port"); // 8080

// 带默认值的真值判断
const enabled = isConfigPathTruthyWithDefaults(
  config,
  "server.debug",
  { "server.debug": false }
);

// 检查命令是否存在
if (hasBinary('node')) {
  console.log('Node.js is available');
}
```