# agents/agent-paths.ts 解读

**文件路径**: `src/agents/agent-paths.ts`

## 文件用途
代理（Agent）路径解析和环境变量设置模块，负责确定代理的工作目录和配置环境变量。

## 主要函数

### resolveOpenClawAgentDir(): string
- **功能**: 解析 OpenClaw 代理目录
- **返回值**: 代理目录路径
- **解析优先级**:
  1. 环境变量 `OPENCLAW_AGENT_DIR`
  2. 环境变量 `PI_CODING_AGENT_DIR`
  3. 默认路径：`<stateDir>/agents/default/agent`
- **实现逻辑**:
  - 检查环境变量覆盖
  - 使用用户路径解析（支持 `~`）
  - 回退到默认路径

### ensureOpenClawAgentEnv(): string
- **功能**: 确保代理环境变量已设置
- **返回值**: 代理目录路径
- **实现逻辑**:
  - 调用 `resolveOpenClawAgentDir` 获取目录
  - 如果未设置 `OPENCLAW_AGENT_DIR`，则设置
  - 如果未设置 `PI_CODING_AGENT_DIR`，则设置
  - 返回目录路径

## 主要依赖
- `node:path` - 路径处理
- `../config/paths.js` - 状态目录解析
- `../routing/session-key.js` - 默认代理 ID
- `../utils.js` - 用户路径解析

## 使用场景
- 确定代理工作目录
- 设置环境变量
- 初始化代理环境
- 路径解析和验证

## 代码行数
26 行

## 重要特性
- 环境变量优先级
- 向后兼容（PI_CODING_AGENT_DIR）
- 用户路径支持
- 默认路径回退

## 环境变量
- `OPENCLAW_AGENT_DIR`: OpenClaw 代理目录覆盖
- `PI_CODING_AGENT_DIR`: 向后兼容的环境变量

## 默认路径结构
```
~/.openclaw/
└── agents/
    └── default/
        └── agent/
            ├── .agent/
            ├── .session/
            └── ...
```

## 使用示例
```typescript
// 解析代理目录
const agentDir = resolveOpenClawAgentDir();
// "/Users/user/.openclaw/agents/default/agent"

// 确保环境变量已设置
ensureOpenClawAgentEnv();
// 设置 OPENCLAW_AGENT_DIR 和 PI_CODING_AGENT_DIR
// 返回代理目录路径

// 使用环境变量覆盖
process.env.OPENCLAW_AGENT_DIR = "/custom/agent/path";
const customDir = resolveOpenClawAgentDir();
// "/custom/agent/path"
```