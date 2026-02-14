# channels/channel-config.ts 解读

**文件路径**: `src/channels/channel-config.ts`

## 文件用途
通道配置匹配和解析模块，提供灵活的键匹配、回退和通配符支持，用于通道配置的查找和解析。

## 主要类型定义

### ChannelMatchSource
通道匹配来源
- `"direct"` - 直接匹配
- `"parent"` - 父级匹配
- `"wildcard"` - 通配符匹配

### ChannelEntryMatch<T>
通道条目匹配结果
- `entry?: T` - 匹配的条目
- `key?: string` - 匹配的键
- `wildcardEntry?: T` - 通配符条目
- `wildcardKey?: string` - 通配符键
- `parentEntry?: T` - 父级条目
- `parentKey?: string` - 父级键
- `matchKey?: string` - 最终匹配的键
- `matchSource?: ChannelMatchSource` - 匹配来源

## 主要函数

### applyChannelMatchMeta<TResult>(result, match): TResult
- **功能**: 应用匹配元数据到结果对象
- **参数**:
  - `result` - 结果对象
  - `match` - 匹配对象
- **返回值**: 应用元数据后的结果对象
- **实现逻辑**:
  1. 检查是否有匹配键和来源
  2. 将 `matchKey` 和 `matchSource` 添加到结果
  3. 返回更新后的结果

### resolveChannelMatchConfig<TEntry, TResult>(match, resolveEntry): TResult | null
- **功能**: 解析通道匹配配置
- **参数**:
  - `match` - 匹配对象
  - `resolveEntry` - 条目解析函数
- **返回值**: 解析后的配置或 null（无匹配）
- **实现逻辑**:
  1. 检查是否有匹配的条目
  2. 调用解析函数处理条目
  3. 应用匹配元数据
  4. 返回结果

### normalizeChannelSlug(value): string
- **功能**: 规范化通道标识符为 URL 友好的 slug
- **参数**: 原始字符串
- **返回值**: 规范化后的 slug
- **实现逻辑**:
  1. 去除前后空白
  2. 转小写
  3. 移除前导 `#`
  4. 非字母数字替换为 `-`
  5. 移除首尾 `-`

### buildChannelKeyCandidates(...keys): string[]
- **功能**: 构建通道键候选列表（去重）
- **参数**: 可变数量的键
- **返回值**: 去重后的键数组
- **实现逻辑**:
  1. 创建 Set 用于去重
  2. 遍历所有键
  3. 过滤无效值和重复值
  4. 返回候选数组

### resolveChannelEntryMatch<T>(params): ChannelEntryMatch<T>
- **功能**: 解析通道条目匹配（直接匹配 + 通配符）
- **参数**:
  - `entries` - 条目映射
  - `keys` - 候选键列表
  - `wildcardKey` - 通配符键
- **返回值**: 匹配结果
- **实现逻辑**:
  1. 遍历候选键查找直接匹配
  2. 查找通配符匹配
  3. 返回匹配结果（可能包含两种匹配）

### resolveChannelEntryMatchWithFallback<T>(params): ChannelEntryMatch<T>
- **功能**: 解析通道条目匹配（支持回退、规范化、父级）
- **参数**:
  - `entries` - 条目映射
  - `keys` - 候选键列表
  - `parentKeys` - 父级键列表
  - `wildcardKey` - 通配符键
  - `normalizeKey` - 键规范化函数
- **返回值**: 匹配结果（含匹配来源）
- **实现逻辑**:
  1. 尝试直接匹配
  2. 尝试规范化键匹配
  3. 尝试父级键匹配
  4. 尝试父级规范化键匹配
  5. 尝试通配符匹配
  6. 返回第一个成功的匹配

### resolveNestedAllowlistDecision(params): boolean
- **功能**: 解析嵌套允许列表决策
- **参数**:
  - `outerConfigured` - 外层是否配置
  - `outerMatched` - 外层是否匹配
  - `innerConfigured` - 内层是否配置
  - `innerMatched` - 内层是否匹配
- **返回值**: 是否允许
- **实现逻辑**:
  1. 外层未配置 → 允许
  2. 外层配置但未匹配 → 拒绝
  3. 外层匹配，内层未配置 → 允许
  4. 内层配置 → 返回内层匹配结果

## 主要依赖
- 无外部依赖（纯工具函数）

## 使用场景
- 通道配置查找
- 允许列表匹配
- 配置回退逻辑
- 键规范化
- 嵌套配置决策

## 代码行数
183 行

## 重要特性
- 多层匹配策略（直接、父级、通配符）
- 键规范化支持
- 回退机制
- 去重处理
- 匹配来源追踪

## 匹配优先级

### resolveChannelEntryMatchWithFallback
1. **直接匹配** - 精确键匹配
2. **规范化直接匹配** - 规范化后的键匹配
3. **父级匹配** - 父级键精确匹配
4. **规范化父级匹配** - 父级键规范化匹配
5. **通配符匹配** - 通配符键匹配

## 使用示例

### 规范化通道标识符
```typescript
normalizeChannelSlug("#General-Chat");
// "general-chat"

normalizeChannelSlug("  Test_Channel  ");
// "test-channel"
```

### 构建候选键
```typescript
const candidates = buildChannelKeyCandidates(
  "user1",
  "user2",
  null,
  "user1", // 重复
  undefined
);
// ["user1", "user2"]
```

### 直接匹配
```typescript
const match = resolveChannelEntryMatch({
  entries: {
    "user1": { allowed: true },
    "*": { allowed: false }
  },
  keys: ["user1"],
  wildcardKey: "*"
});
// {
//   entry: { allowed: true },
//   key: "user1",
//   wildcardEntry: { allowed: false },
//   wildcardKey: "*"
// }
```

### 回退匹配
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: {
    "general": { enabled: true },
    "*": { enabled: false }
  },
  keys: ["#general", "general"],
  parentKeys: ["channel"],
  wildcardKey: "*",
  normalizeKey: normalizeChannelSlug
});
// {
//   entry: { enabled: true },
//   key: "general",
//   matchKey: "general",
//   matchSource: "direct"
// }
```

### 父级匹配
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: {
    "channel": { defaultPermissions: "read" }
  },
  keys: ["#subchannel"],
  parentKeys: ["channel"]
});
// {
//   entry: { defaultPermissions: "read" },
//   key: "channel",
//   parentEntry: { defaultPermissions: "read" },
//   parentKey: "channel",
//   matchKey: "channel",
//   matchSource: "parent"
// }
```

### 通配符匹配
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: {
    "*": { allowed: false }
  },
  keys: ["unknown"],
  wildcardKey: "*"
});
// {
//   entry: { allowed: false },
//   key: "*",
//   matchKey: "*",
//   matchSource: "wildcard"
// }
```

### 嵌套允许列表决策
```typescript
// 外层未配置
resolveNestedAllowlistDecision({
  outerConfigured: false,
  outerMatched: false,
  innerConfigured: false,
  innerMatched: false
});
// true

// 外层匹配，内层匹配
resolveNestedAllowlistDecision({
  outerConfigured: true,
  outerMatched: true,
  innerConfigured: true,
  innerMatched: true
});
// true

// 外层未匹配
resolveNestedAllowlistDecision({
  outerConfigured: true,
  outerMatched: false,
  innerConfigured: true,
  innerMatched: true
});
// false
```

### 解析配置
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: {
    "user1": { permissions: ["read", "write"] },
    "*": { permissions: ["read"] }
  },
  keys: ["user1"],
  wildcardKey: "*"
});

const config = resolveChannelMatchConfig(match, (entry) => {
  return {
    permissions: entry.permissions,
    matchKey: match.matchKey,
    matchSource: match.matchSource
  };
});
// {
//   permissions: ["read", "write"],
//   matchKey: "user1",
//   matchSource: "direct"
// }
```

## 应用场景

### Discord 频道权限
```typescript
// 配置
const permissions = {
  "general": { allow: ["read", "write"] },
  "channel-*": { allow: ["read"] },
  "*": { allow: [] }
};

// 查找
const match = resolveChannelEntryMatchWithFallback({
  entries: permissions,
  keys: ["#general"],
  wildcardKey: "*"
});
// 直接匹配 "general"
```

### WhatsApp 群组路由
```typescript
// 配置
const routing = {
  "family-group": { agent: "family" },
  "*": { agent: "default" }
};

// 查找
const match = resolveChannelEntryMatchWithFallback({
  entries: routing,
  keys: ["family-group"],
  wildcardKey: "*"
});
// 使用 "family" 代理
```

### Telegram 账户映射
```typescript
// 配置
const accounts = {
  "123456789": { name: "User 1" },
  "*": { name: "Unknown" }
};

// 查找
const match = resolveChannelEntryMatch({
  entries: accounts,
  keys: ["123456789"],
  wildcardKey: "*"
});
// 找到账户 "User 1"
```

## 性能考虑
- 使用 `Object.prototype.hasOwnProperty` 检查属性（更安全）
- 提前返回（找到第一个匹配即停止）
- Set 去重（O(1) 查找）
- 键规范化缓存（可在外部实现）

## 错误处理
- 无显式错误抛出
- 返回 null 或部分匹配
- 优雅降级（通配符兜底）

## 扩展性

### 自定义规范化
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: config,
  keys: ["#Test-Channel"],
  normalizeKey: (key) => {
    // 自定义逻辑
    return key.toLowerCase().replace(/[^a-z0-9]/g, "");
  }
});
```

### 多层回退
```typescript
const match = resolveChannelEntryMatchWithFallback({
  entries: config,
  keys: ["specific"],
  parentKeys: ["general", "default"], // 多个父级
  wildcardKey: "*"
});
```

## 嵌套决策逻辑

### 规则
| 外层配置 | 外层匹配 | 内层配置 | 内层匹配 | 结果 |
|---------|---------|---------|---------|------|
| false   | -       | -       | -       | true |
| true    | false   | -       | -       | false |
| true    | true    | false   | -       | true |
| true    | true    | true    | true    | true |
| true    | true    | true    | false   | false |

### 典型用法
```typescript
// 通道级别 + 群组级别
const allowed = resolveNestedAllowlistDecision({
  outerConfigured: !!config.channel.allowFrom,
  outerMatched: isAllowed(config.channel.allowFrom, userId),
  innerConfigured: !!config.channel.groups?.[groupId]?.allowFrom,
  innerMatched: isAllowed(config.channel.groups?.[groupId]?.allowFrom, userId)
});
```

## 设计模式
- **策略模式**: 多种匹配策略
- **责任链模式**: 依次尝试各种匹配
- **模板方法**: 解析流程框架
