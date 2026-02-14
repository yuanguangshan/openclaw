# infra/home-dir.ts 解读

**文件路径**: `src/infra/home-dir.ts`

## 文件用途
主目录解析模块，负责在不同操作系统上解析用户主目录路径，支持环境变量、显式设置和 `~` 前缀展开。

## 主要函数

### resolveEffectiveHomeDir(env?, homedir?): string | undefined
- **功能**: 解析有效的主目录路径
- **参数**:
  - `env` - 环境变量对象（默认：`process.env`）
  - `homedir` - 主目录函数（默认：`os.homedir`）
- **返回值**: 解析后的主目录绝对路径或 undefined
- **实现逻辑**:
  1. 调用 `resolveRawHomeDir` 获取原始路径
  2. 使用 `path.resolve` 转换为绝对路径
  3. 返回规范化后的路径

### resolveRawHomeDir(env, homedir): string | undefined
- **功能**: 解析原始主目录路径
- **参数**:
  - `env` - 环境变量对象
  - `homedir` - 主目录函数
- **返回值**: 原始路径字符串或 undefined
- **实现逻辑**:
  1. 检查 `OPENCLAW_HOME` 环境变量
  2. 如果是以 `~` 开头，展开为实际主目录
  3. 如果 `OPENCLAW_HOME` 不存在，检查 `HOME`（Unix/Linux）
  4. 如果 `HOME` 不存在，检查 `USERPROFILE`（Windows）
  5. 最后尝试 `os.homedir()`
  6. 返回第一个找到的有效路径

### normalizeSafe(homedir): string | undefined
- **功能**: 安全调用主目录函数
- **参数**: 主目录函数
- **返回值**: 规范化后的路径或 undefined
- **实现逻辑**:
  1. 调用函数获取路径
  2. 捕获异常并返回 undefined
  3. 确保不会因异常而失败

### resolveRequiredHomeDir(env?, homedir?): string
- **功能**: 解析必需的主目录路径（必须有值）
- **参数**:
  - `env` - 环境变量对象
  - `homedir` - 主目录函数
- **返回值**: 主目录路径（保证非空）
- **实现逻辑**:
  1. 调用 `resolveEffectiveHomeDir`
  2. 如果为 undefined，返回当前工作目录
  3. 确保始终返回有效路径

### expandHomePrefix(input, opts?): string
- **功能**: 展开输入字符串中的 `~` 前缀
- **参数**:
  - `input` - 输入字符串
  - `opts` - 选项
    - `home` - 显式主目录
    - `env` - 环境变量
    - `homedir` - 主目录函数
- **返回值**: 展开后的字符串
- **实现逻辑**:
  1. 检查是否以 `~` 开头
  2. 如果不是，直接返回
  3. 解析主目录（优先级：opts.home > resolveEffectiveHomeDir）
  4. 替换 `~` 或 `~/` 或 `~\` 为实际路径
  5. 返回展开后的字符串

## 辅助函数

### normalize(value): string | undefined
- **功能**: 规范化字符串
- **参数**: 任意字符串
- **返回值**: 去除空白后的字符串或 undefined
- **实现逻辑**: 去除前后空白，空字符串返回 undefined

## 主要依赖
- `node:os` - 操作系统信息（`os.homedir`）
- `node:path` - 路径处理（`path.resolve`）

## 使用场景
- 解析用户配置目录
- 跨平台路径处理
- `~` 前缀展开
- 环境变量解析
- 测试和模拟

## 代码行数
78 行

## 重要特性
- 跨平台支持（Unix/Linux/macOS/Windows）
- 灵活的环境变量优先级
- `~` 前缀展开
- 安全的错误处理
- 可注入依赖（便于测试）

## 优先级顺序

### 主目录解析
1. `OPENCLAW_HOME` - OpenClaw 专用环境变量
2. `HOME` - Unix/Linux/macOS 标准
3. `USERPROFILE` - Windows 标准
4. `os.homedir()` - Node.js API

### 展开前缀
1. `opts.home` - 显式提供
2. `resolveEffectiveHomeDir` - 自动解析

## 跨平台支持

### Unix/Linux/macOS
```bash
# 标准环境变量
HOME=/home/user
```

### Windows
```bash
# 标准环境变量
USERPROFILE=C:\Users\user

# 兼容
HOME=C:\Users\user
```

### 通用
```bash
# OpenClaw 专用
OPENCLAW_HOME=/custom/path
OPENCLAW_HOME=~/custom
```

## 使用示例

### 基本用法
```typescript
// 使用默认环境
const home = resolveEffectiveHomeDir();
// macOS: "/Users/username"
// Linux: "/home/username"
// Windows: "C:\Users\username"
```

### 使用 OPENCLAW_HOME
```typescript
process.env.OPENCLAW_HOME = "/custom/openclaw";
const home = resolveEffectiveHomeDir();
// "/custom/openclaw"
```

### 使用 `~` 前缀
```typescript
process.env.OPENCLAW_HOME = "~/my-openclaw";
const home = resolveEffectiveHomeDir();
// "/Users/username/my-openclaw"
```

### 展开路径
```typescript
expandHomePrefix("~/Documents/file.txt");
// "/Users/username/Documents/file.txt"

expandHomePrefix("~", { home: "/custom" });
// "/custom"

expandHomePrefix("/absolute/path");
// "/absolute/path" (不变)
```

### 必需主目录
```typescript
// 如果所有方法都失败，使用当前目录
const home = resolveRequiredHomeDir();
// 保证返回有效路径
```

### 自定义环境
```typescript
const customEnv = {
  HOME: "/custom/home",
  USERPROFILE: undefined
};

const home = resolveEffectiveHomeDir(customEnv);
// "/custom/home"
```

### 测试和模拟
```typescript
// 模拟主目录函数
const mockHomedir = () => "/mock/home";
const home = resolveEffectiveHomeDir(undefined, mockHomedir);
// "/mock/home"

// 模拟环境变量
const home = resolveEffectiveHomeDir({
  HOME: "/test/home"
});
// "/test/home"
```

## 错误处理

### `os.homedir()` 失败
- 捕获异常
- 返回 undefined
- 不抛出错误

### 无有效主目录
- `resolveEffectiveHomeDir` 返回 undefined
- `resolveRequiredHomeDir` 返回当前工作目录

### 无效 `OPENCLAW_HOME`
- `~` 前缀需要有效的基础主目录
- 返回 undefined

## 路径规范化

### `path.resolve` 效果
```typescript
// 相对路径
resolveEffectiveHomeDir({ HOME: "relative" });
// "/current/working/dir/relative"

// 绝对路径
resolveEffectiveHomeDir({ HOME: "/absolute" });
// "/absolute"

// `.` 和 `..`
resolveEffectiveHomeDir({ HOME: "/path/to/.././dir" });
// "/path/dir"
```

## 展开规则

### `~` 前缀模式
- `~` → 主目录
- `~/` → 主目录 + `/`
- `~\` → 主目录 + `\`（Windows）
- `~anything` → 保持不变（仅匹配 `~` 或 `~/` 或 `~\`）

### 示例
```typescript
expandHomePrefix("~");
// "/home/user"

expandHomePrefix("~/file.txt");
// "/home/user/file.txt"

expandHomePrefix("~subdir/file.txt");
// "~subdir/file.txt" (不变)
```

## 测试策略

### 单元测试
```typescript
// 测试环境变量
expect(resolveEffectiveHomeDir({ HOME: "/test" })).toBe("/test");

// 测试主目录函数
expect(resolveEffectiveHomeDir(undefined, () => "/mock")).toBe("/mock");

// 测试展开
expect(expandHomePrefix("~/file", { home: "/custom" })).toBe("/custom/file");
```

### 集成测试
```typescript
// 测试实际环境
const home = resolveEffectiveHomeDir();
expect(home).toBeTruthy();
expect(path.isAbsolute(home)).toBe(true);
```

## 性能考虑
- 早期返回（找到第一个有效值）
- 避免不必要的 `path.resolve` 调用
- 缓存建议：在应用启动时调用一次并缓存结果

## 安全考虑
- 路径规范化防止目录遍历
- 使用 `path.resolve` 确保绝对路径
- 验证输入路径（在外部调用者）

## 兼容性

### Node.js 版本
- `os.homedir()` 在所有现代 Node.js 版本中可用
- 环境变量跨平台兼容

### 操作系统
- **Unix/Linux/macOS**: 优先使用 `HOME`
- **Windows**: 优先使用 `USERPROFILE`，支持 `HOME` 作为后备

## 扩展性

### 自定义优先级
```typescript
// 可以轻松修改优先级顺序
function resolveRawHomeDir(env, homedir) {
  // 添加新的环境变量
  const custom = normalize(env.CUSTOM_HOME);
  if (custom) return custom;
  
  // 原有逻辑...
}
```

### 自定义展开
```typescript
// 可以添加更多前缀
function expandHomePrefix(input, opts) {
  // 扩展 `~` 逻辑
  if (input.startsWith("~")) {
    // ...
  }
  
  // 添加其他前缀
  if (input.startsWith("$HOME/")) {
    const home = resolveEffectiveHomeDir(opts?.env);
    return input.replace("$HOME/", home + "/");
  }
  
  return input;
}
```

## 典型用法流程

### OpenClaw 配置加载
```
1. resolveEffectiveHomeDir() → 获取主目录
2. path.join(home, ".openclaw") → 配置目录
3. path.join(configDir, "openclaw.json") → 配置文件
```

### 用户提供路径
```
1. 用户输入: "~/Documents/config"
2. expandHomePrefix(input) → 展开路径
3. path.resolve(expanded) → 绝对路径
```

## 相关文件
- `../config/paths.js` - 配置路径解析
- `../config/config.js` - 配置加载
- `../utils.js` - 通用工具函数
