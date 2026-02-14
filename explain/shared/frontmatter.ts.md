# shared/frontmatter.ts 解读

**文件路径**: `src/shared/frontmatter.ts`

## 文件用途
处理 Markdown 文件的前置元数据（frontmatter），提供解析、验证和标准化功能。

## 主要函数

### normalizeStringList(input: unknown): string[]
- **功能**: 将输入标准化为字符串数组
- **参数**: 任意类型的输入
- **返回值**: 标准化后的字符串数组
- **实现逻辑**:
  - 空值 → 空数组
  - 数组 → 映射为字符串并去空
  - 字符串 → 按逗号分割并去空
  - 其他 → 空数组

### getFrontmatterString(frontmatter: Record<string, unknown>, key: string): string | undefined
- **功能**: 从 frontmatter 对象获取字符串值
- **参数**:
  - `frontmatter`: frontmatter 对象
  - `key`: 键名
- **返回值**: 字符串值或 undefined
- **实现**: 仅返回字符串类型的值

### parseFrontmatterBool(value: string | undefined, fallback: boolean): boolean
- **功能**: 解析布尔值字符串，支持回退值
- **参数**:
  - `value`: 待解析的字符串
  - `fallback`: 解析失败时的默认值
- **返回值**: 布尔值
- **实现逻辑**:
  - 尝试解析字符串
  - 解析失败返回默认值

### resolveOpenClawManifestBlock(params: { frontmatter: Record<string, unknown>; key?: string }): Record<string, unknown> | undefined
- **功能**: 解析 OpenClaw 清单块
- **参数**:
  - `frontmatter`: frontmatter 对象
  - `key`: 清单块键名（默认 "metadata"）
- **返回值**: 解析后的清单对象或 undefined
- **实现逻辑**:
  1. 获取清单字符串
  2. 使用 JSON5 解析
  3. 查找清单键（支持新旧键名）
  4. 返回清单对象

## 主要依赖
- `json5` - JSON5 解析器（支持注释和宽松语法）
- `../compat/legacy-names.js` - 旧版名称兼容性
- `../utils/boolean.js` - 布尔值解析

## 使用场景
- Markdown 文件元数据处理
- 技能（skill）和插件（plugin）配置解析
- 代理（agent）配置文件处理
- 文档前置信息提取

## 代码行数
61 行

## 重要特性
- 支持 JSON5 格式（允许注释和宽松语法）
- 兼容新旧键名
- 灵活的字符串列表处理
- 类型安全的布尔值解析

## 前置数据示例
```markdown
---
metadata: '{
  "openclaw": {
    "name": "My Skill",
    "version": "1.0.0",
    "enabled": true,
    "permissions": ["read", "write"]
  }
}'
tags: "automation, productivity"
---

# My Skill Documentation
```

## 使用示例
```typescript
const frontmatter = {
  metadata: JSON.stringify({
    openclaw: {
      name: "My Skill",
      version: "1.0.0"
    }
  }),
  tags: "automation, productivity",
  enabled: "true"
};

// 解析清单
const manifest = resolveOpenClawManifestBlock({
  frontmatter,
  key: "metadata"
});

// 解析标签列表
const tags = normalizeStringList(frontmatter.tags);
// ["automation", "productivity"]

// 解析布尔值
const enabled = parseFrontmatterBool(
  frontmatter.enabled as string,
  false
);
// true
```