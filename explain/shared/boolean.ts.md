# shared/boolean.ts 解读

**文件路径**: `src/shared/boolean.ts`

## 文件用途
提供布尔值解析和转换的工具函数，用于处理各种格式的布尔值输入。

## 主要函数

### isTruthyValue(value: unknown): boolean
- **功能**: 判断值是否为真值
- **参数**: 任意类型的值
- **返回值**: 布尔值
- **实现逻辑**: 支持多种真值格式（"true"、"1"、"yes"、"on" 等）

### parseBoolean(value: unknown): boolean | undefined
- **功能**: 解析布尔值字符串
- **参数**: 任意类型的值
- **返回值**: 解析后的布尔值或 undefined（如果无法解析）

## 使用场景
- 配置文件中的布尔选项解析
- 环境变量转换
- 用户输入验证
- CLI 参数处理

## 测试覆盖
包含对应的单元测试文件 `boolean.test.ts`

## 代码行数
约 40 行

## 主要特点
- 支持多种常见布尔值表示形式
- 对无效输入返回 undefined 而非抛出错误
- 类型安全，接受 unknown 类型参数