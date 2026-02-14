# commands/gateway-status.ts 解读

**文件路径**: `src/commands/gateway-status.ts`

## 文件用途
网关状态检查命令模块，检查网关服务的健康状态、连接状态和性能指标。

## 主要类型定义

### GatewayStatusCheckResult
网关状态检查结果：
- `status`: 状态类型（ok/warning/error/critical）
- `message`: 状态消息
- `details`: 状态详情对象

### GatewayHealthCheck
网关健康检查结果：
- `gatewayStatus`: 网关连接状态
- `channelsHealth`: 各通道的健康状态
- `lastProbe`: 最后探测时间和结果
- `error`: 错误信息

## 主要函数

### getGatewayStatusCheck(cfg, runtime, options?): Promise<GatewayStatusCheck>
- **功能**: 检查网关状态
- **参数**:
  - `cfg`: OpenClaw 配置
  - `runtime`: 运行时环境
  - `options`: 选项（超时、详细模式等）
- **实现逻辑**:
  1. 检查网关配置
  2. 测试网关连接
  3. 检查各通道健康状态
  4. 测量性能指标
 5. 生成状态报告

## 主要依赖
- `../config/config.js` - OpenClaw 配置
- `../runtime.js` - 运行时环境
- `../cli/progress.js` - 进度显示
- `../gateway/call.js` - 网关调用
- `../channels/plugins/index.js` - 通道注册表
- `../channels/plugins/types.js` - 通道类型
- `../terminal/health-style.js` - 健康样式

## 使用场景
- 网关健康监控
- 网关连接检查
- 性能监控
- 问题诊断
- 状态报告

## 代码行数
408 行

## 重要特性
- 实时状态检查
- 多通道健康分析
- 性能指标监控
- 详细的状态报告
- 错误诊断
- 超时控制

## 状态类型

### status: 状态
- `ok` - 正常运行
- `warning` - 有警告但可用
- `error` - 错误
- `critical` - 严重错误

### gatewayStatus: 网关状态
- `connected` - 已连接
- `disconnected` - 未连接
- `starting` - 启动中
- `stopping` - 停止中
- `stopped` - 已停止

### channelsHealth: 通道健康状态
- `ok`: 正常
- `degraded`: 降级
- `error`: 错误

## 使用示例
```bash
# 检查网关状态
openclaw gateway-status

# 详细模式
openclaw gateway-status --verbose

# 设置超时（毫秒）
openclaw gateway-status --timeout 30000

# JSON 输出
openclaw gateway-status --json
```