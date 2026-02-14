# commands/health.ts 解读

**文件路径**: `src/commands/health.ts`

## 文件用途
健康检查命令模块，提供系统级健康检查，包括网关状态、通道连接、代理状态、会话统计等。

## 主要类型定义

### ChannelAccountHealthSummary
通道账户健康摘要：
- `accountId`: 账户 ID
- `configured`: 是否已配置
- `linked`: 是否已链接
- `authAgeMs`: 认证年龄（毫秒）
- `probe`: 探测结果
- `lastProbeAt`: 最后探测时间

### ChannelHealthSummary
通道健康摘要，继承自 ChannelAccountHealthSummary，添加：
- `accounts`: 账户摘要映射

### AgentHeartbeatSummary
代理心跳摘要：
- `everyMs`: 心跳间隔
- `lastAt`: 最后心跳时间
- `lastError`: 最后错误

### AgentHealthSummary
代理健康摘要：
- `agentId`: 代理 ID
- `name`: 代理名称
- `isDefault`: 是否为默认代理
- `heartbeat`: 心跳摘要
- `sessions`: 会话健康摘要

### HealthSummary
完整健康摘要：
- `ok`: 总体状态（总是 true）
- `ts`: 时间戳
- `durationMs`: 执行耗时
- `channels`: 通道健康摘要映射
- `channelOrder`: 通道顺序
- `channelLabels`: 通道标签映射
- `heartbeatSeconds`: 默认代理心跳秒数
- `defaultAgentId`: 默认代理 ID
- `agents`: 代理健康摘要数组
- `sessions`: 会话摘要
  - `path`: 会话存储路径
  - `count`: 会话总数
  - `recent`: 最近 5 个会话
- `key`: 会话键
  - `updatedAt`: 更新时间
  - `age`: 年龄（毫秒）

## 主要函数

### formatDurationParts(ms: number): string
- **功能**: 格式化时长（毫秒）为可读格式
- **参数**: 毫秒数
- **返回值**: 格式化后的字符串（如 "2d 3h 45m"）

### resolveHeartbeatSummary(cfg, agentId)
- **功能**: 解析代理心跳摘要
- **参数**: 配置和代理 ID
- **返回值**: 心跳摘要对象

### resolveAgentOrder(cfg)
- **功能**: 解析代理顺序
- **参数**: OpenClaw 配置
- **返回值**: 代理顺序（default 代理总是在最前）

### buildSessionSummary(storePath)
- **功能**: 构建会话摘要
- **参数**: 会话存储路径
- **返回值**: 会话摘要对象

### isAccountEnabled(account): boolean
- **功能**: 检查账户是否启用
- **参数**: 账户对象
- **返回值**: 是否启用

### formatProbeLine(probe, opts): string | null
- **功能**: 格式化探测结果行
- **参数**:
  - `probe`: 探测结果
  - `opts`: 选项（botUsernames 数组）
- **返回值**: 格式化的探测行或 null

### formatAccountProbeTiming(summary): string | null
- **功能**: 格式化账户探测时机
- **参数**: 通道账户健康摘要
- **返回值**: 格式化的时机字符串

### isProbeFailure(summary): boolean
- **功能**: 判断探测是否失败
- **参数**: 通道账户健康摘要
- **返回值**: 是否失败

### formatHealthChannelLines(summary, opts): string[]
- **功能**: 格式化健康通道行
- **参数**:
  - `summary`: 健康摘要
  - `opts`: 选项（accountMode、accountIdsByChannel）
- **返回值**: 格式化的行数组

## 主要依赖
- `../config/config.js` - OpenClaw 配置
- `../runtime.js` - 运行时环境
- `../agents/agent-scope.js` - 代理作用域
- `../channels/plugins/types.js` - 通道类型
- `../channels/plugins/index.js` - 通道注册表
- `../channels/plugins/helpers.js` - 通道工具
- `../cli/progress.js` - 进度显示
- `../loadConfig` - 配置加载
- `../config/sessions.js` - 会话配置
- `../gateway/call.js` - 网关调用
- `../gateway/protocol/index.js` - 网关协议
- `../infra/heartbeat-runner.js` - 心跳运行器
- `../routing/bindings.js` - 路由绑定
- `../routing/session-key.js` - 会话键
- `../terminal/health-style.js` - 健康样式
- `../terminal/theme.js` - 终端主题

## 使用场景
- 系统健康检查
- 通道状态监控
- 代理状态查看
- 会话状态统计
- 性能监控

## 代码行数
751 行

## 重要特性
- 完整的系统状态快照
- 多通道健康检查
- 代理心跳监控
- 时间格式化显示
- 颜色编码（富文本支持）
- 会话统计和排序

## 输出格式示例
```
Health Summary (12.3456s)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Channels:
  telegram: abc123 (ok) - 23ms ago
  discord: ok - configured, linked
  slack: ok - configured
```

## 代理状态示例
```
Agents:
  default: MyBot (ok) - heartbeat: 45s ago
  agent1: TaskBot (ok) - heartbeat: 120s ago
```

## 会话统计示例
```
Sessions:
  Path: /path/to/sessions.json
  Count: 15
  Recent:
    abc123: 2h ago
    def456: 3d ago
```

## 时间格式化规则
- ≥ 7 天: 显示为 XwXd
- ≥ 1 天: 显示为 Xdh
- ≥ 1 小时: 显示为 Xm
- ≥ 1 分钟: 显示为 Xm
- 小于 1 分钟: 显示为 Xm
```