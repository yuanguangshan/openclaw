# commands/sessions.ts 解读

**文件路径**: `src/commands/sessions.ts`

## 文件用途
会话列表命令模块，显示和管理 OpenClaw 的会话状态和统计信息。

## 主要类型定义

### SessionRow
会话行数据：
- `key`: 会话键
- `kind`: 会话类型（direct/group/global/unknown）
- `updatedAt`: 更新时间戳
- `ageMs`: 距今时间（毫秒）
- `sessionId`: 会话 ID
- `systemSent`: 是否已发送系统消息
- `abortedLastRun`: 上次运行是否中止
- `thinkingLevel`: 思考级别
- `verboseLevel`: 详细级别
- `reasoningLevel`: 推理级别
- `elevatedLevel`: 提升级别
- `responseUsage`: 响应使用类型
- `groupActivation`: 群组激活状态
- `inputTokens`: 输入 Token 数量
- `outputTokens`: 输出 Token 数量
- `totalTokens`: 总 Token 数量
- `totalTokensFresh`: 是否为新的总 Token 统计
- `model`: 模型
- `contextTokens`: 上下文 Token 数量

## 主要函数

### formatKTokens(value: number): string
- **功能**: 格式化 Token 数量（以千为单位）
- **参数**: Token 数量
- **返回值**: 格式化后的字符串

### truncateKey(key: string): string
- **功能**: 截断过长的会话键（显示前6个字符...后6个字符）
- **参数**: 会话键
- **返回值**: 截断后的键

### colorByPct(label, pct, rich): string
- **功能**: 根据百分比返回带颜色的标签
- **参数**:
  - `label`: 标签文本
  - `pct`: 百分比（0-100）
  - `rich`: 是否启用颜色
- **返回值**: 带颜色标签

### formatTokensCell(total, contextTokens, rich)
- **功能**: 格式化 Token 单元格
- **参数**:
  - `total`: 总 Token 数
  - `contextTokens`: 上下文 Token 数
  - `rich`: 是否启用颜色
- **返回值**: 格式化的单元格

### formatKindCell(kind, rich): string
- **功能**: 格式化会话类型单元格
- **参数**:
  - `kind`: 会话类型
  - `rich`: 是否启用颜色
- **返回值**: 格式化的单元格

### formatAgeCell(updatedAt, rich): string
- **功能**: 格式化年龄单元格
- **参数**:
  - `updatedAt`: 更新时间
  - `rich`: 是否启用颜色
- **返回值**: 格式化的单元格

### formatModelCell(model, rich): string
- **功能**: 格式化模型单元格
- **参数**:
  - `model`: 模型标识
  - `rich`: 是否启用颜色
- **返回值**: 格式化的单元格

### formatFlagsCell(row, rich): string
- **功能**: 格式化标志单元格
- **参数**:
  - `row`: 会话行数据
  - `rich`: 是否启用颜色
- **返回值**: 格式化的标志字符串

### toRows(store): SessionRow[]
- **功能**: 将会话存储转换为行数组
- **参数**: 会话存储对象
- **返回值**: 会话行数组（按更新时间排序）

### sessionsCommand(opts, runtime)
- **功能**: 执行会话列表命令
- **参数**:
  - `opts`: 命令选项
    - `json`: JSON 输出
    - `store`: 存储路径
    - `active`: 活跃分钟数（过滤条件）
  - `runtime`: 运行时环境
- **实现逻辑**:
  1. 加载配置
  2. 解析模型和上下文 Token
  3. 加载会话存储
  4. 按活跃时间过滤会话
  5. 根据输出格式显示结果
  6. 显示表头和会话行

## 主要依赖
- `../runtime.js` - 运行时类型
- `../agents/context.js` - 上下文 Token
- `../agents/defaults.js` - 默认值
- `../agents/model-selection.js` - 模型选择
- `../config/config.js` - 配置管理
- `../config/sessions.js` - 会话配置
- `../infra/format-time/format-relative.ts` - 时间格式化
- `../terminal/theme.js` - 终端主题

## 使用场景
- 查看所有会话
- 监控会话活动状态
- Token 使用统计
- 会话清理和管理

## 代码行数
282 行

## 重要特性
- 智能过滤（活跃会话）
- Token 统计和显示
- 彩色支持
- 时间格式化（如 2m, 1h, 3d）
- 会话类型分类

## 输出格式

### JSON 输出
```json
{
  "path": "/path/to/sessions.json",
  "count": 10,
  "activeMinutes": 5,
  "sessions": [...]
}
```

### 文本输出（表格）
```
Kind     Key                         Age         Model            Tokens (ctx %)   Flags
direct  abc123...xyz              2m      gpt-4             1500/2000 (75%)      think:high
group   group:xyz               30m     claude-3          5000/6000 (83%)     verbose:full
global  global                      1d      gpt-4             1000/? (?)         system
```

## 使用示例
```bash
# 列出所有会话
openclaw sessions

# 只显示最近 5 分钟内活跃的会话
openclaw sessions --active 5

# 使用特定存储
openclaw sessions --store /path/to/custom.json

# JSON 输出
openclaw sessions --json
```

## 常量定义
- `KIND_PAD`: 类型列宽度（6）
- `KEY_PAD`: 键列宽度（26）
- `AGE_PAD`: 年龄列宽度（9）
- `MODEL_PAD`: 模型列宽度（14）
- `TOKENS_PAD`: Token 列宽度（20）

## 颜色规则
- 百分比 ≥ 95%: 红色（错误）
- 百分比 ≥ 80%: 黄色（警告）
- 百分比 ≥ 60%: 绿色（成功）
- 百分比 < 60%: 灰色（静音）