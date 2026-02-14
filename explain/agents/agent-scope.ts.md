# agents/agent-scope.ts 解读

**文件路径**: `src/agents/agent-scope.ts`

## 文件用途
代理（Agent）配置和作用域解析模块，提供代理 ID 解析、配置合并和路径管理功能。

## 主要类型定义

### AgentEntry
代理配置条目类型（从 OpenClawConfig 提取）

### ResolvedAgentConfig
解析后的代理配置：
- `name`: 代理名称
- `workspace`: 工作空间路径
- `agentDir`: 代理目录
- `model`: 模型配置
- `skills`: 技能列表
- `memorySearch`: 记忆搜索配置
- `humanDelay`: 人工延迟配置
- `heartbeat`: 心跳配置
- `identity`: 身份配置
- `groupChat`: 群聊配置
- `subagents`: 子代理配置
- `sandbox`: 沙箱配置
- `tools`: 工具配置

## 主要函数

### listAgents(cfg: OpenClawConfig): AgentEntry[]
- **功能**: 列出配置中的所有代理
- **参数**: OpenClaw 配置
- **返回值**: 代理条目数组
- **实现**: 过滤并验证代理列表

### listAgentIds(cfg: OpenClawConfig): string[]
- **功能**: 列出所有代理 ID（去重）
- **参数**: OpenClaw 配置
- **返回值**: 代理 ID 数组
- **实现逻辑**:
  - 列出所有代理
  - 标准化代理 ID
  - 去重处理
  - 无代理时返回默认 ID

### resolveDefaultAgentId(cfg: OpenClawConfig): string
- **功能**: 解析默认代理 ID
- **参数**: OpenClaw 配置
- **返回值**: 默认代理 ID
- **实现逻辑**:
  - 无代理时返回默认 ID
  - 查找标记为 default 的代理
  - 多个 default 时使用第一个并警告
  - 无 default 时使用第一个

### resolveSessionAgentIds(params): { defaultAgentId: string; sessionAgentId: string }
- **功能**: 解析会话的代理 ID
- **参数**:
  - `sessionKey`: 会话键
  - `config`: OpenClaw 配置
- **返回值**: 包含默认代理 ID 和会话代理 ID 的对象
- **实现逻辑**:
  - 解析默认代理 ID
  - 从会话键解析代理 ID
  - 会话 ID 优先于默认 ID

### resolveSessionAgentId(params): string
- **功能**: 解析会话代理 ID（便捷函数）
- **参数**: 会话键和配置
- **返回值**: 会话代理 ID

### resolveAgentEntry(cfg: OpenClawConfig, agentId: string): AgentEntry | undefined
- **功能**: 查找代理配置条目
- **参数**: 配置和代理 ID
- **返回值**: 代理条目或 undefined

### resolveAgentConfig(cfg: OpenClawConfig, agentId: string): ResolvedAgentConfig | undefined
- **功能**: 解析代理配置
- **参数**: 配置和代理 ID
- **返回值**: 解析后的配置对象
- **实现**: 提取并规范化所有配置字段

### resolveAgentSkillsFilter(cfg: OpenClawConfig, agentId: string): string[] | undefined
- **功能**: 解析代理技能过滤器
- **参数**: 配置和代理 ID
- **返回值**: 技能名称数组或 undefined

### resolveAgentModelPrimary(cfg: OpenClawConfig, agentId: string): string | undefined
- **功能**: 解析代理主模型
- **参数**: 配置和代理 ID
- **返回值**: 主模型 ID
- **支持格式**: 字符串或 `{ primary: string, fallbacks: string[] }`

### resolveAgentModelFallbacksOverride(cfg: OpenClawConfig, agentId: string): string[] | undefined
- **功能**: 解析代理模型回退覆盖
- **参数**: 配置和代理 ID
- **返回值**: 回退模型数组或 undefined
- **特殊处理**: 空数组表示禁用全局回退

### resolveAgentWorkspaceDir(cfg: OpenClawConfig, agentId: string): string
- **功能**: 解析代理工作空间目录
- **参数**: 配置和代理 ID
- **返回值**: 工作空间目录路径
- **解析优先级**:
  1. 代理配置的 workspace
  2. 默认代理的全局默认 workspace
  3. 默认代理工作空间
  4. 状态目录下的 `workspace-{id}`

### resolveAgentDir(cfg: OpenClawConfig, agentId: string): string
- **功能**: 解析代理目录
- **参数**: 配置和代理 ID
- **返回值**: 代理目录路径
- **解析优先级**:
  1. 代理配置的 agentDir
  2. 状态目录下的 `agents/{id}/agent`

## 主要依赖
- `node:path` - 路径处理
- `../config/config.js` - 配置类型
- `../config/paths.js` - 状态目录
- `../routing/session-key.js` - 会话键解析
- `../utils.js` - 用户路径
- `./workspace.js` - 默认工作空间

## 使用场景
- 代理配置管理
- 多代理支持
- 路径解析
- 配置合并和继承

## 代码行数
193 行

## 重要特性
- 多代理支持
- 配置继承
- 路径解析优先级
- 去重和标准化
- 默认值处理

## 配置示例
```typescript
const config = {
  agents: {
    list: [
      {
        id: "agent1",
        name: "Primary Agent",
        default: true,
        workspace: "/projects/my-project",
        model: {
          primary: "gpt-4",
          fallbacks: ["gpt-3.5-turbo"]
        },
        skills: ["coding", "web"]
      },
      {
        id: "agent2",
        name: "Secondary Agent"
      }
    ],
    defaults: {
      workspace: "/default/workspace"
    }
  }
};

// 解析默认代理 ID
const defaultId = resolveDefaultAgentId(config);
// "agent1"

// 解析代理配置
const agentConfig = resolveAgentConfig(config, "agent1");
// {
//   name: "Primary Agent",
//   workspace: "/projects/my-project",
//   model: { primary: "gpt-4", fallbacks: [...] },
//   skills: ["coding", "web"],
//   ...
// }

// 解析主模型
const primaryModel = resolveAgentModelPrimary(config, "agent1");
// "gpt-4"

// 解析工作空间目录
const workspace = resolveAgentWorkspaceDir(config, "agent1");
// "/projects/my-project"
```