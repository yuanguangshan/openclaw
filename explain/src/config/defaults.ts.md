# 文件解释：src/config/defaults.ts

## 文件路径
`src/config/defaults.ts`

## 文件用途
配置默认值应用模块，负责为配置对象应用各种默认值和标准化处理。

## 主要类/函数

### `DEFAULT_MODEL_ALIASES`
模型别名映射表，将简短别名映射到完整的模型 ID：
- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

### `applyMessageDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用消息默认值，特别是 `ackReactionScope` 默认为 `"group-mentions"`。

### `applySessionDefaults(cfg: OpenClawConfig, options?: SessionDefaultsOptions): OpenClawConfig`
应用会话默认值：
- 将 `session.mainKey` 标准化为 `"main"`（主会话总是使用 "main" 键）
- 如果用户设置了其他值，发出警告

### `applyTalkApiKey(config: OpenClawConfig): OpenClawConfig`
应用 Talk API 密钥，从环境变量解析并添加到配置中（如果配置中尚未设置）。

### `applyModelDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用模型默认值：
- 标准化 `reasoning` 字段为布尔值
- 默认 `input` 类型为 `["text"]`
- 设置默认成本（input、output、cacheRead、cacheWrite 均为 0）
- 设置默认上下文窗口（DEFAULT_CONTEXT_TOKENS）
- 设置默认最大 token 数（不超过上下文窗口）
- 为模型别名添加 `alias` 属性

### `applyAgentDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用 Agent 默认值：
- 默认 `maxConcurrent` 为 `DEFAULT_AGENT_MAX_CONCURRENT`（4）
- 默认 `subagents.maxConcurrent` 为 `DEFAULT_SUBAGENT_MAX_CONCURRENT`（8）

### `applyLoggingDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用日志默认值，设置 `logging.redactSensitive` 为 `"tools"`。

### `applyContextPruningDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用上下文修剪默认值：
- 根据认证模式（api_key 或 oauth）设置不同的默认值
- 默认 `contextPruning.mode` 为 `"cache-ttl"`，`ttl` 为 `"1h"`
- OAuth 模式下 `heartbeat.every` 为 `"1h"`，API key 模式下为 `"30m"`
- API key 模式下，Anthropic 模型的 `cacheRetention` 默认为 `"short"`

### `applyCompactionDefaults(cfg: OpenClawConfig): OpenClawConfig`
应用压缩默认值，设置 `compaction.mode` 为 `"safeguard"`。

### `resolveAnthropicDefaultAuthMode(cfg: OpenClawConfig): AnthropicAuthDefaultsMode | null`
解析 Anthropic 默认认证模式，检查配置的认证配置文件和环境变量，确定是使用 API key 还是 OAuth。

### `resetSessionDefaultsWarningForTests()`
重置会话默认值警告状态（用于测试）。

## 类型定义

```typescript
type SessionDefaultsOptions = {
  warn?: (message: string) => void;
  warnState?: WarnState;
};

type AnthropicAuthDefaultsMode = "api_key" | "oauth";
```

## 主要依赖

- `./types.js` - 配置类型
- `./types.models.js` - 模型配置类型
- `./agent-limits.js` - Agent 并发限制默认值
- `./talk.js` - Talk API 密钥解析
- `../agents/defaults.js` - Agent 默认上下文窗口
- `../agents/model-selection.js` - 模型引用解析

## 重要逻辑说明

1. **分层默认值**：不同的配置部分有各自的默认值应用函数，按顺序调用。

2. **模型别名处理**：在应用模型默认值时，会为模型别名添加 `alias` 属性，便于后续处理。

3. **认证模式推断**：根据配置的认证配置文件和环境变量推断应该使用的认证模式，影响上下文修剪和心跳间隔的默认值。

4. **警告机制**：会话键警告使用 `warnState` 跟踪，确保只发出一次警告。

5. **不可变更新**：所有默认值应用函数都返回新对象，不修改原配置。

## 行数统计
471 行
