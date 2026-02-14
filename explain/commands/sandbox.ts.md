# commands/sandbox.ts 解读

**文件路径**: `src/commands/sandbox.ts`

## 文件用途
沙箱管理命令模块，提供 Docker 容器和浏览器的列表、创建和删除功能。

## 主要类型定义

### SandboxListOptions
列表选项：
- `browser`: 是否包含浏览器
- `json`: JSON 输出

### SandboxRecreateOptions
重新创建选项：
- `all`: 操作所有容器
- `session`: 仅特定会话的容器
- `agent`: 仅特定代理的容器
- `browser`: 包含浏览器
- `force`: 强制删除（不确认）

### FilteredContainers
过滤后的容器：
- `containers`: 容器信息数组
- `browsers`: 浏览器信息数组

### ContainerItem
容器项类型（容器或浏览器）

## 主要命令

### sandboxListCommand(opts, runtime)
- **功能**: 列出沙箱容器和浏览器
- **参数**: 列表选项
- **实现逻辑**:
  1. 根据选项加载容器和浏览器列表
  2. 如果 JSON 模式，输出 JSON 格式
  3. 否则显示格式化的列表
  4. 显示统计摘要

### sandboxRecreateCommand(opts, runtime)
- **功能**: 重新创建沙箱容器
- **参数**: 重新创建选项
- **实现逻辑**:
  1. 验证选项（必须指定目标）
  2. 过滤匹配的容器和浏览器
  3. 如果没有匹配项，显示提示并返回
  4. 显示预览信息
  5. 确认删除操作
  6. 删除匹配的容器
  7. 显示删除结果
  8. 如果有失败则退出

## 主要函数

### validateRecreateOptions(opts, runtime): boolean
- **功能**: 验证重新创建选项
- **验证规则**:
  - 至少需要一个目标（all、session 或 agent）
  - 不能同时指定多个互斥的目标
  - 浏览器选项必须与 all 一起使用
- **返回值**: 是否有效

### fetchAndFilterContainers(opts): Promise<FilteredContainers>
- **功能**: 获取并过滤容器
- **参数**: 重新创建选项
- **返回值**: 过滤后的容器和浏览器列表
- **过滤逻辑**:
  - `all`: 包含所有容器和浏览器
  - `session`: 仅匹配会话键的容器和浏览器
  - `agent`: 仅匹配代理前缀的容器和浏览器
  - `browser`: 排除容器

### confirmRecreate(): Promise<boolean>
- **功能**: 确认删除操作
- **返回值**: 是否继续

### removeContainers(filtered, runtime): Promise<{ successCount, failCount }>
- **功能**: 删除容器
- **参数**:
  - `filtered`: 过滤后的容器
  - `runtime`: 运行时环境
- **返回值**: 删除结果统计

### removeContainer(containerName, removeFn, runtime): Promise<{ success }>
- **功能**: 删除单个容器
- **参数**:
  - `containerName`: 容器名称
  - `removeFn`: 删除函数
  - `runtime`: 运时环境
- **返回值**: 删除结果

### createAgentMatcher(agentId): (item) => boolean
- **功能**: 创建代理匹配器函数
- **实现**: 检查会话键或容器名是否以代理 ID 为前缀

## 主要依赖
- `../agents/sandbox.js` - 沙箱核心功能
- `../runtime.js` - 运行时环境
- `@clack/prompts` - 提示库
- `./sandbox-display.js` - 显示功能

## 使用场景
- 清理不再使用的容器
- 重置沙箱环境
- 故障排除
- 容器管理

## 代码行数
201 行

## 重要特性
- 多种过滤选项
- 容器和浏览器统一管理
- 删除操作确认
- 失败统计和报告
- JSON 和文本双输出模式

## 容器和浏览器类型

### 容器
- 完整的 Docker 容器
- 包含会话键标识

### 浏览器
- 轻量级浏览器（如 Firefox）
- 会话关联
- 支持 Screenshot API

## 会话键格式
- 容器：`session:{sessionKey}:{agentId}`
- 浏览器：`session:{sessionKey}:{browserId}`

## 使用示例
```bash
# 列出所有容器
openclaw sandbox list

# 列出容器和浏览器
openclaw sandbox list --browser

# JSON 输出
openclaw sandbox list --json

# 重新创建所有容器
openclaw sandbox recreate --all

# 重新创建特定会话的容器
openclaw sandbox recreate --session abc123

# 重新创建特定代理的容器
openclaw sandbox recreate --agent agent1

# 强制删除（不确认）
openclaw sandbox recreate --all --force

# 包含浏览器
openclaw sandbox recreate --all --browser
```

## 验证错误示例

```bash
# 缺少目标
openclaw sandbox recreate
# 错误：Please specify --all, --session <key>, or --agent <id>

# 多个互斥目标
openclaw sandbox recreate --session abc --agent xyz
# 错误：Please specify only one of: --all, --session, or --agent

# 浏览器选项错误
openclaw sandbox recreate --session abc
# 错误：Please specify --all, --session <key>, or --agent <id> with --browser
```

## 输出示例

### JSON 模式
```json
{
  "containers": [...],
  "browsers": [...]
}
```

### 文本模式
```
Containers:
  session:abc123:agent1  Running  gpt-4
  session:abc124:agent2  Exited

Browsers:
  session:abc123:firefox  Running
```

## 删除结果
```
✓ Removed session:abc123:agent1
✓ Removed session:abc123:firefox
✗ Failed to remove session:abc124:agent2: Error: container not found
```

## 安全考虑
- 删除操作需要确认（除非 --force）
- 容器删除会停止运行中的容器
- 数据丢失风险：会话关联的数据可能受影响