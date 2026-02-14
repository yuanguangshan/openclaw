# gateway/hooks-mapping.ts 解读

**文件路径**: `src/gateway/hooks-mapping.ts`

## 文件用途
Webhook 映射规则引擎，负责解析配置中的映射规则，根据请求上下文匹配并应用相应的转换，将外部请求转换为 OpenClaw 动作。

## 主要类型定义

### HookMappingResolved
解析后的映射规则
- `id: string` - 映射 ID
- `matchPath?: string` - 路径匹配模式
- `matchSource?: string` - 来源匹配模式
- `action: "wake" | "agent"` - 动作类型
- `wakeMode?: "now" | "next-heartbeat"` - 唤醒模式
- `name?: string` - Webhook 名称
- `agentId?: string` - 目标代理 ID
- `sessionKey?: string` - 会话键（支持模板）
- `messageTemplate?: string` - 消息模板
- `textTemplate?: string` - 文本模板
- `deliver?: boolean` - 是否投递
- `allowUnsafeExternalContent?: boolean` - 是否允许不安全内容
- `channel?: HookMessageChannel` - 消息通道
- `to?: string` - 目标地址（支持模板）
- `model?: string` - 模型（支持模板）
- `thinking?: string` - 思考级别（支持模板）
- `timeoutSeconds?: number` - 超时秒数
- `transform?: HookMappingTransformResolved` - 转换函数配置

### HookMappingTransformResolved
转换函数配置
- `modulePath: string` - 模块路径
- `exportName?: string` - 导出函数名

### HookMappingContext
映射上下文
- `payload: Record<string, unknown>` - 请求负载
- `headers: Record<string, string>` - 请求头
- `url: URL` - 完整 URL
- `path: string` - 请求路径

### HookAction
生成的动作
- **Wake**: `{ kind: "wake", text: string, mode: "now" | "next-heartbeat" }`
- **Agent**: `{ kind: "agent", message, name?, agentId?, wakeMode, sessionKey?, deliver?, allowUnsafeExternalContent?, channel?, to?, model?, thinking?, timeoutSeconds? }`

### HookMappingResult
映射结果
- 成功: `{ ok: true, action: HookAction }`
- 跳过: `{ ok: true, action: null, skipped: true }`
- 失败: `{ ok: false, error: string }`

## 主要函数

### resolveHookMappings(hooks?, opts?): HookMappingResolved[]
- **功能**: 解析和规范化所有 Webhook 映射规则
- **参数**:
  - `hooks` - Webhook 配置
  - `opts` - 选项（configDir）
- **返回值**: 解析后的映射规则数组
- **实现逻辑**:
  1. 收集所有映射规则（自定义 + 预设）
  2. 应用预设映射（如 Gmail）
  3. 解析转换目录路径
  4. 标准化每个映射规则
  5. 返回规范化后的映射数组

### applyHookMappings(mappings, ctx): Promise<HookMappingResult | null>
- **功能**: 应用映射规则，将请求转换为动作
- **参数**:
  - `mappings` - 映射规则数组
  - `ctx` - 请求上下文
- **返回值**: Promise，结果或 null（无匹配）
- **实现逻辑**:
  1. 遍历所有映射规则
  2. 检查规则是否匹配当前请求
  3. 从映射构建基础动作
  4. 如果有转换函数，加载并应用转换
  5. 合并基础动作和转换结果
  6. 验证最终动作
  7. 返回第一个匹配的结果

### normalizeHookMapping(mapping, index, transformsDir): HookMappingResolved
- **功能**: 标准化单个映射规则
- **参数**:
  - `mapping` - 原始映射配置
  - `index` - 索引（用于生成默认 ID）
  - `transformsDir` - 转换目录
- **返回值**: 标准化后的映射规则
- **实现逻辑**:
  1. 生成或使用提供的 ID
  2. 标准化匹配路径和来源
  3. 设置默认动作类型
  4. 解析转换函数配置
  5. 返回完整规范化对象

### mappingMatches(mapping, ctx): boolean
- **功能**: 检查映射规则是否匹配请求
- **参数**:
  - `mapping` - 映射规则
  - `ctx` - 请求上下文
- **返回值**: 是否匹配
- **实现逻辑**:
  1. 检查路径匹配（如果配置了）
  2. 检查来源匹配（如果配置了）
  3. 返回匹配结果

### buildActionFromMapping(mapping, ctx): HookMappingResult
- **功能**: 从映射规则构建基础动作
- **参数**:
  - `mapping` - 映射规则
  - `ctx` - 请求上下文
- **返回值**: 动作结果或错误
- **实现逻辑**:
  1. 根据 action 类型构建不同动作
  2. 渲染模板（使用上下文数据）
  3. 返回构建的动作对象

### mergeAction(base, override, defaultAction): HookMappingResult
- **功能**: 合并基础动作和转换覆盖
- **参数**:
  - `base` - 基础动作
  - `override` - 转换覆盖
  - `defaultAction` - 默认动作类型
- **返回值**: 合并后的动作结果
- **实现逻辑**:
  1. 确定动作类型（优先级：override > base > default）
  2. 根据类型合并字段
  3. 验证合并结果
  4. 返回最终动作

### validateAction(action): HookMappingResult
- **功能**: 验证动作有效性
- **参数**: 动作对象
- **返回值**: 验证结果
- **实现逻辑**:
  1. Wake 动作必须包含 text
  2. Agent 动作必须包含 message
  3. 返回验证结果

### loadTransform(transform): Promise<HookTransformFn>
- **功能**: 加载转换函数模块
- **参数**: 转换配置
- **返回值**: Promise，转换函数
- **实现逻辑**:
  1. 检查缓存
  2. 动态导入模块
  3. 解析导出函数（支持 default、transform、指定导出名）
  4. 缓存并返回函数

### renderTemplate(template, ctx): string
- **功能**: 渲染模板字符串
- **参数**:
  - `template` - 模板字符串
  - `ctx` - 上下文
- **返回值**: 渲染后的字符串
- **实现逻辑**:
  1. 查找所有 `{{ expr }}` 占位符
  2. 解析表达式并获取值
  3. 转换为字符串
  4. 返回渲染结果

### resolveTemplateExpr(expr, ctx): unknown
- **功能**: 解析模板表达式
- **参数**:
  - `expr` - 表达式字符串
  - `ctx` - 上下文
- **返回值**: 表达式的值
- **支持的表达式**:
  - `path` - 请求路径
  - `now` - 当前时间 ISO 字符串
  - `headers.xxx` - 请求头
  - `query.xxx` - 查询参数
  - `payload.xxx` - 负载字段
  - `xxx` - 负载字段（简写）

### getByPath(input, pathExpr): unknown
- **功能**: 从对象获取嵌套值（支持点路径和数组索引）
- **参数**:
  - `input` - 输入对象
  - `pathExpr` - 路径表达式
- **返回值**: 值或 undefined
- **实现逻辑**:
  1. 解析路径表达式（支持 `a.b.c[0].d` 格式）
  2. 逐层访问对象
  3. 返回最终值或 undefined

## 主要依赖
- `node:path` - 路径处理
- `node:url` - URL 处理
- `../config/config.js` - 配置类型和路径
- `./hooks.js` - Webhook 类型

## 使用场景
- Gmail 集成（预设映射）
- 第三方服务 Webhook 接收
- 自定义请求转换逻辑
- 复杂的请求路由和处理
- 动态内容生成

## 代码行数
474 行

## 重要特性
- 灵活的匹配规则（路径、来源）
- 模板系统（支持嵌套字段访问）
- 转换函数支持（动态模块加载）
- 预设映射（Gmail 等）
- 模块缓存优化
- 路径安全检查（防止目录遍历）
- 转换合并策略（覆盖 vs 默认）

## 预设映射

### Gmail
```typescript
{
  id: "gmail",
  match: { path: "gmail" },
  action: "agent",
  wakeMode: "now",
  name: "Gmail",
  sessionKey: "hook:gmail:{{messages[0].id}}",
  messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
}
```

## 模板表达式示例

### 简单字段
```
{{ payload.field }}
{{ payload.user.name }}
{{ payload.items[0].id }}
```

### 特殊表达式
```
{{ path }}              // /hooks/gmail
{{ now }}                // 2025-02-15T10:30:00.000Z
{{ headers.content-type }} // application/json
{{ query.token }}        // abc123
{{ payload.source }}     // gmail
```

### 复杂嵌套
```
{{ payload.messages[0].from.email }}
{{ payload.data.items[2].metadata.tags[0] }}
```

## 转换函数示例

### 模块导出
```typescript
// hooks/transforms/custom-transform.ts
export function transform(ctx: HookMappingContext) {
  // 自定义逻辑
  return {
    kind: "agent",
    message: `Custom: ${ctx.payload.message}`,
    agentId: "custom-agent",
  };
}

// 或
export default function(ctx: HookMappingContext) {
  // ...
}

// 或
export { transform as customName };
```

### 配置
```typescript
{
  hooks: {
    mappings: [{
      id: "custom",
      match: { path: "custom" },
      action: "agent",
      transform: {
        module: "custom-transform.ts",
        export: "customName"  // 可选，默认 default 或 transform
      }
    }]
  }
}
```

## 配置示例

### 基础映射
```typescript
{
  hooks: {
    enabled: true,
    token: "secret",
    mappings: [
      {
        id: "slack-mention",
        match: { path: "slack/mention" },
        action: "agent",
        name: "Slack",
        sessionKey: "hook:slack:{{payload.user}}",
        messageTemplate: "Slack mention from {{payload.user}}: {{payload.text}}",
        channel: "slack",
        to: "{{payload.channel}}",
      },
      {
        id: "webhook-wake",
        match: { path: "wake" },
        action: "wake",
        textTemplate: "Wake up! Triggered at {{now}}",
        wakeMode: "now",
      }
    ]
  }
}
```

### 使用预设
```typescript
{
  hooks: {
    enabled: true,
    token: "secret",
    presets: ["gmail"],
    gmail: {
      allowUnsafeExternalContent: true,  // 覆盖 Gmail 预设配置
    }
  }
}
```

### 自定义转换目录
```typescript
{
  hooks: {
    enabled: true,
    token: "secret",
    transformsDir: "./custom-transforms",  // 相对于配置目录
    mappings: [{
      id: "custom",
      action: "agent",
      transform: {
        module: "my-transform.js",  // 在 custom-transforms/ 中
      }
    }]
  }
}
```

## 安全特性
- 转换模块必须在配置目录内（防止目录遍历）
- 转换函数必须是函数类型
- 路径规范化（移除前后斜杠）
- 严格的输入验证

## 性能优化
- 转换函数缓存（避免重复加载）
- 提前返回（第一个匹配的映射）
- 路径解析优化

## 错误处理
- 转换模块加载失败
- 转换函数类型错误
- 路径越界错误
- 必需字段缺失（text、message）
