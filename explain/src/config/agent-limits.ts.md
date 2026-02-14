# 文件解释：src/config/agent-limits.ts

## 文件路径
`src/config/agent-limits.ts`

## 文件用途
Agent 并发限制常量和解析工具。

## 主要类/函数

### `DEFAULT_AGENT_MAX_CONCURRENT`
默认 Agent 最大并发数：4。

### `DEFAULT_SUBAGENT_MAX_CONCURRENT`
默认子 Agent 最大并发数：8。

### `resolveAgentMaxConcurrent(cfg?: OpenClawConfig): number`
解析 Agent 最大并发数：
- 从配置中读取 `agents.defaults.maxConcurrent`
- 如果是有效数字，返回 `Math.max(1, Math.floor(raw))`
- 否则返回默认值 4

### `resolveSubagentMaxConcurrent(cfg?: OpenClawConfig): number`
解析子 Agent 最大并发数：
- 从配置中读取 `agents.defaults.subagents.maxConcurrent`
- 如果是有效数字，返回 `Math.max(1, Math.floor(raw))`
- 否则返回默认值 8

## 主要依赖

- `./types.js` - 配置类型

## 重要逻辑说明

1. **最小值保护**：确保并发数至少为 1，防止设置为 0 或负数。

2. **向下取整**：使用 `Math.floor` 确保并发数为整数。

3. **配置缺失处理**：当配置缺失或无效时，使用硬编码的默认值。

## 行数统计
21 行
