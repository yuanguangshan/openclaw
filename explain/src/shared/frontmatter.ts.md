# 文件解释：src/shared/frontmatter.ts

## 文件路径
`src/shared/frontmatter.ts`

## 文件用途
Frontmatter（文件元数据）解析工具，用于处理 Markdown 文件等格式的元数据。

## 主要类/函数

### `normalizeStringList(input: unknown): string[]`
将输入标准化为字符串列表：
- `null` 或 `undefined` 返回空数组
- 数组：每个元素转为字符串，去除空格，过滤空值
- 字符串：按逗号分割，去除空格，过滤空值
- 其他类型返回空数组

### `getFrontmatterString(frontmatter: Record<string, unknown>, key: string): string | undefined`
从 frontmatter 对象中获取字符串值，如果值不是字符串则返回 `undefined`。

### `parseFrontmatterBool(value: string | undefined, fallback: boolean): boolean`
解析 frontmatter 中的布尔值，使用 `parseBooleanValue` 解析，失败时使用默认值。

### `resolveOpenClawManifestBlock(params): Record<string, unknown> | undefined`
解析 OpenClaw 清单元数据块。从 frontmatter 中获取指定键（默认为 "metadata"），解析 JSON5 格式的配置，并查找清单键。

**逻辑**：
1. 从 frontmatter 中获取指定键的字符串值
2. 使用 JSON5 解析
3. 查找清单键（MANIFEST_KEY 或 LEGACY_MANIFEST_KEYS）
4. 返回清单对象或 `undefined`

## 主要依赖

- `json5` - JSON5 解析器
- `../compat/legacy-names.js` - 旧版名称常量
- `../utils/boolean.js` - 布尔值解析

## 重要逻辑说明

1. **灵活的字符串列表**：`normalizeStringList` 支持数组或逗号分隔的字符串，方便配置。

2. **清单键查找**：`resolveOpenClawManifestBlock` 支持多个清单键名称，包括新版和旧版键名，确保向后兼容。

3. **错误容错**：所有解析函数在遇到无效输入时都返回安全的默认值（空数组、`undefined` 等），不会抛出异常。

4. **JSON5 支持**：使用 JSON5 解析器，支持注释、尾随逗号等扩展 JSON 语法。

## 行数统计
61 行
