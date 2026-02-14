# config/agent-dirs.ts 解读

**文件路径**: `src/config/agent-dirs.ts`

## 文件用途
代理（Agent）目录管理模块，负责解析和验证多代理配置中的代理目录设置，防止目录冲突。

## 主要类型定义

### DuplicateAgentDir
重复代理目录对象：
- `agentDir`: 代理目录路径
- `agentIds`: 使用该目录的代理 ID 数组

### DuplicateAgentDirError
重复代理目录错误类：
- 继承自 Error
- 包含 `duplicates` 属性，存储所有重复的代理目录

## 主要函数

### canonicalizeAgentDir(agentDir: string): string
- **功能**: 规范化代理目录路径
- **参数**: 代理目录路径
- **返回值**: 规范化后的路径
- **实现逻辑**:
  - 解析为绝对路径
  - macOS 和 Windows 平台转换为小写（不区分大小写）

### collectReferencedAgentIds(cfg: OpenClawConfig): string[]
- **功能**: 收集配置中所有引用的代理 ID
- **参数**: OpenClaw 配置对象
- **返回值**: 代理 ID 数组
- **实现逻辑**:
  1. 收代理列表中的默认代理 ID
  2. 收集所有代理列表中的 ID
  3. 收集所有绑定中的代理 ID
  4. 返回去重后的 ID 列表

### resolveEffectiveAgentDir(cfg: OpenClawConfig, agentId: string, deps?): string
- **功能**: 解析代理的有效目录
- **参数**:
  - `cfg`: OpenClaw 配置
  - `agentId`: 代理 ID
  - `deps`: 依赖项（环境变量、主目录函数）
- **返回值**: 解析后的代理目录路径
- **实现逻辑**:
  1. 检查代理配置中的显式 `agentDir` 设置
  2. 如果没有显式设置，使用默认路径：`<stateDir>/agents/<id>/agent`
  3. 支持用户路径解析（如 `~`）

### findDuplicateAgentDirs(cfg: OpenClawConfig, deps?): DuplicateAgentDir[]
- **功能**: 查找重复的代理目录
- **参数**:
  - `cfg`: OpenClaw 配置
  - `deps`: 依赖项
- **返回值**: 重复代理目录数组
- **实现逻辑**:
  1. 遍历所有引用的代理 ID
  2. 解析每个代理的有效目录
  3. 使用规范化路径作为键进行分组
  4. 返回有多个代理共享的目录

### formatDuplicateAgentDirError(dups: DuplicateAgentDir[]): string
- **功能**: 格式化重复代理目录错误信息
- **参数**: 重复代理目录数组
- **返回值**: 格式化的错误信息字符串
- **内容**:
  - 错误描述
  - 冲突列表
  - 修复建议

## 主要依赖
- `node:os` - 操作系统信息
- `node:path` - 路径处理
- `../infra/home-dir.js` - 主目录解析
- `../routing/session-key.js` - 代理 ID 标准化
- `../utils.js` - 路径工具函数
- `./paths.js` - 状态目录解析

## 使用场景
- 多代理配置验证
- 代理目录冲突检测
- 配置启动前检查
- 错误诊断和修复提示

## 代码行数
113 行

## 重要特性
- 跨平台路径规范化
- 配置冲突检测
- 清晰的错误提示
- 灵活的目录配置

## 错误信息示例
```
Duplicate agentDir detected (multi-agent config).
Each agent must have a unique agentDir; sharing it causes auth/session state collisions and token invalidation.

Conflicts:
- /home/user/.openclaw/agents/default: "agent1", "agent2"

Fix: remove the shared agents.list[].agentDir override (or give each agent its own directory).
If you want to share credentials, copy auth-profiles.json instead of sharing entire agentDir.
```