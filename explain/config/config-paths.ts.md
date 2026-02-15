# config/config-paths.ts 解读

**文件路径**: `src/config/config-paths.ts`

## 文件用途
配置路径解析模块，提供点号表示法的配置路径解析和操作功能。

## 主要类型定义

### PathNode
路径节点类型（递归对象）

## 主要函数

### parseConfigPath(raw): { ok: boolean; path?: string[]; error?: string }
- **功能**: 解析配置路径
- **参数**: 路径字符串（点号表示法）
- **返回值**: 解析结果
- **实现逻辑**:
  1. 去除空白
  2. 按 `.` 分割
  3. 验证每个部分非空
  4. 检查阻止键（`__proto__`, `prototype`, `constructor`）
  5. 返回路径数组或错误

### setConfigValueAtPath(root, path, value): void
- **功能**: 设置配置路径上的值
- **参数**:
  - `root` - 根节点
  - `path` - 路径数组
  - `value` - 要设置的值
- **实现逻辑**:
  1. 遍历路径创建中间对象
  2. 设置最终键的值

### unsetConfigValueAtPath(root, path): boolean
- **功能**: 删除配置路径上的值
- **参数**:
  - `root` - 根节点
  - `path` - 路径数组
- **返回值**: 是否成功删除
- **实现逻辑**:
  1. 遍历路径记录栈
  2. 验证中间对象存在
  3. 删除最终键
  4. 清理空对象

### getConfigValueAtPath(root, path): unknown
- **功能**: 获取配置路径上的值
- **参数**:
  - `root` - 根节点
  - `path` - 路径数组
- **返回值**: 路径上的值或 undefined
- **实现逻辑**:
  1. 遍历路径
  2. 验证中间对象是纯对象
  3. 返回最终值

## 辅助函数

### isPlainObject(value): boolean
- **功能**: 检查是否为纯对象
- **实现逻辑**: 非数组和对象类型

## 主要依赖
- `../utils.js` - 工具函数

## 使用场景
- 嵌套配置访问
- 配置文件解析
- 动态配置修改
- 配置验证

## 代码行数
84 行

## 重要特性
- 点号表示法支持
- 阻止危险键
- 安全的嵌套访问
- 空对象清理

## 路径示例

### 解析路径
```typescript
parseConfigPath("agents.defaults.contextTokens")
// { ok: true, path: ["agents", "defaults", "contextTokens"] }

parseConfigPath("")
// { ok: false, error: "Invalid path. Use dot notation (e.g. foo.bar)" }

parseConfigPath("__proto__")
// { ok: false, error: "Invalid path segment." }
```

### 设置值
```typescript
const config = { agents: {} };

setConfigValueAtPath(config, ["agents", "defaults", "contextTokens"], 8192);
// config = { agents: { defaults: { contextTokens: 8192 } } }
```

### 删除值
```typescript
const config = { agents: { defaults: { contextTokens: 8192 } } };

unsetConfigValueAtPath(config, ["agents", "defaults", "contextTokens"]);
// config = { agents: { defaults: {} } }
```

### 获取值
```typescript
const config = { agents: { defaults: { contextTokens: 8192 } } };

getConfigValueAtPath(config, ["agents", "defaults", "contextTokens"]);
// 8192

getConfigValueAtPath(config, ["agents", "defaults", "missing"]);
// undefined

getConfigValueAtPath(config, ["agents", "__proto__", "pollute"]);
// undefined（路径不安全）
```

## 阻止键

以下键被阻止以防止原型污染：
- `__proto__`
- `prototype`
- `constructor`

## 安全考虑

### 防止原型污染
- 阻止危险键
- 验证纯对象类型

### 路径验证
- 每个部分非空
- 点号分隔符

### 空对象清理
- 删除后清理空对象
- 防止内存泄漏

## 使用模式

### 动态配置更新
```typescript
const config = loadConfig();
const { path } = parseConfigPath("agents.defaults.model");
if (path) {
  setConfigValueAtPath(config, path, "anthropic/claude-opus-4-6");
  saveConfig(config);
}
```

### 安全的嵌套访问
```typescript
const config = loadConfig();
const { path } = parseConfigPath("channels.telegram.botToken");
const token = path ? getConfigValueAtPath(config, path) : undefined;
```
