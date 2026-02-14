# utils/shell-argv.ts 解读

**文件路径**: `src/utils/shell-argv.ts`

## 文件用途
Shell 命令行参数解析工具，模拟 Shell 的参数分割逻辑，正确处理引号、转义字符和空格。

## 主要常量

### DOUBLE_QUOTE_ESCAPES
双引号中需要转义的字符集合：`["\\", '"', "$", "`", "\n", "\r"]`

## 主要函数

### isDoubleQuoteEscape(next: string | undefined): next is string
- **功能**: 类型守卫，判断字符是否需要双引号转义
- **参数**: 下一个字符
- **返回值**: 布尔值（类型守卫）

### splitShellArgs(raw: string): string[] | null
- **功能**: 将 Shell 命令字符串分割为参数数组
- **参数**: 原始命令字符串
- **返回值**: 参数数组或 null（解析失败时）
- **实现逻辑**:
  1. 遍历字符串，跟踪状态：
     - `inSingle`: 是否在单引号内
     - `inDouble`: 是否在双引号内
     - `escaped`: 是否转义状态
  2. 根据状态处理每个字符：
     - 转义字符 `\`: 按上下文处理
     - 单引号 `'`: 切换单引号状态
     - 双引号 `"`: 切换双引号状态
     - 空格: 分隔参数（不在引号内）
     - 其他字符: 添加到当前缓冲区
  3. 验证结束状态（无未闭合的引号或转义）
  4. 返回参数数组

## 解析规则

### 单引号 `'`
- 完全字面，所有字符按原样处理
- 不支持转义
- 示例: `'hello world'` → `["hello world"]`

### 双引号 `"`
- 支持转义
- 可转义: `\`, `"`, `$`, `` ` ``, 换行符
- 示例: `"hello \"world\""` → `['hello "world"']`

### 转义字符 `\`
- 在引号外: 转义下一个字符
- 在双引号内: 仅转义特定字符
- 在单引号内: 无特殊含义
- 示例: `hello\ world` → `["hello world"]`

### 空格分隔
- 仅在引号外作为分隔符
- 引号内的空格保留
- 示例: `hello world` → `["hello", "world"]`

## 使用场景
- Shell 命令解析
- 参数分割
- 用户输入处理
- 命令行工具集成

## 代码行数
75 行

## 重要特性
- 完整的引号支持
- 转义字符处理
- 错误检测
- 标准 Shell 兼容

## 解析示例

### 基本参数
```typescript
splitShellArgs("hello world");
// ["hello", "world"]
```

### 单引号
```typescript
splitShellArgs("'hello world' 'foo bar'");
// ["hello world", "foo bar"]
```

### 双引号
```typescript
splitShellArgs('"hello world" "foo bar"');
// ["hello world", "foo bar"]
```

### 转义
```typescript
splitShellArgs('hello\\ world');
// ["hello world"]

splitShellArgs('"hello \\"world\\""');
// ['hello "world"']
```

### 混合
```typescript
splitShellArgs('arg1 "arg 2" \'arg 3\' arg\\ 4');
// ["arg1", "arg 2", "arg 3", "arg 4"]
```

### 错误情况
```typescript
// 未闭合的单引号
splitShellArgs("'hello");
// null

// 未闭合的双引号
splitShellArgs('"hello');
// null

// 未完成的转义
splitShellArgs("hello\\");
// null
```