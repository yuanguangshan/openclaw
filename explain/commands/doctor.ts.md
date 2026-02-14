# commands/doctor.ts 解读

**文件路径**: `src/commands/doctor.ts`

## 文件用途
OpenClaw 诊断命令主入口，执行全面系统检查和配置修复。

## 主要函数

### doctorCommand(runtime, options)
- **功能**: 执行 OpenClaw 诊断
- **参数**:
  - `runtime`: 运行时环境
  - `options`: 诊断选项
- **检查项目**:
  1. 更新检查和提示
  2. UI 协议版本修复
  3. 源安装问题检查
  4. 废弃环境变量警告
  5. 配置加载和迁移
  6. 网关模式检查
  7. 网关认证配置
  8. 旧版状态迁移
  9. 状态完整性检查
  10. 沙箱镜像修复
  11. 沙箱作用域警告
  12. 额外网关服务扫描
  13. 网关服务配置修复
  14. macOS LaunchAgent 覆盖检查
  15. macOS Launchctl 网关环境覆盖检查
  16. 安全警告
  17. Gmail Hooks 模型检查
  18. 工作空间状态检查
  19. Shell 补全检查
  20. 网关健康检查
  21. 网关守护进程修复
  22. 工作空间备份提示

## 主要依赖
- `../runtime.js` - 运行时类型
- `../config/config.js` - 配置管理
- `../agents/agent-scope.js` - 代理作用域
- `../agents/defaults.js` - 默认值
- `../agents/model-catalog.js` - 模型目录
- `../agents/model-selection.js` - 模型选择
- `../cli/command-format.js` - CLI 格式化
- `../config/config.js` - 配置文件操作
- `../config/logging.js` - 配置日志
- `../daemon/service.js` - 网关服务
- `../gateway/auth.js` - 网关认证
- `../gateway/call.js` - 网关调用
- `../infra/openclaw-root.js` - 包根目录
- `../runtime.js` - 运行时
- `../terminal/note.js` - 终端提示
- `../terminal/prompt-style.js` - 提示样式
- `../utils.js` - 工具函数
- `./doctor-auth.js` - 诊断：认证
- `./doctor-completion.js` - 诊断：Shell 补全
- `./doctor-config-flow.js` - 诊断：配置流程
- `./doctor-gateway-daemon-flow.js` - 诊断：网关守护进程
- `./doctor-gateway-health.js` - 诊断：网关健康
- `./doctor-gateway-services.js` - 诊断：网关服务
- `./doctor-install.js` - 诊断：安装
- `./doctor-platform-notes.js` - 诊断：平台注意事项
- `./doctor-prompter.js` - 诊断：提示器
- `./doctor-sandbox.js` - 诊断：沙箱
- `./doctor-security.js` - 诊断：安全
- `./doctor-state-integrity.js` - 诊断：状态完整性
- `./doctor-state-migrations.js` - 诊断：状态迁移
- `./doctor-ui.js` - 诊断：UI
- `./doctor-update.js` - 诊断：更新
- `./doctor-workspace-status.js` - 诊断：工作空间状态
- `./doctor-workspace.js` - 诊断：工作空间
- `./onboard-helpers.js` - 入驻帮助
- `./systemd-linger.js` - Systemd Lingering

## 使用场景
- 系统健康检查
- 配置问题诊断
- 环境问题检测
- 性能问题排查
- 升级前检查

## 代码行数
314 行

## 重要特性
- 全面的系统检查
- 自动修复建议
- 交互式问题解决
- 配置验证和迁移
- 跨平台支持

## 诊断检查类型

### 1. 配置和认证
- 网关模式检查
- 认证配置验证
- 认证配置文件健康检查
- Anthropic OAuth 配置文件 ID 修复
- 废弃 CLI 配置文件移除

### 2. 状态和迁移
- 旧版状态迁移检测
- 状态完整性检查
- 会话状态验证

### 3. 网关和服务
- 网关健康检查
- 网关守护进程修复
- 额外网关服务扫描
- 网关服务配置修复
- Tailscale 集成检查

### 4. 平台特定
- macOS LaunchAgent 覆盖检查
- macOS Launchctl 环境覆盖检查
- Linux Systemd 用户 lingering 检查

### 5. 安全和沙箱
- 安全警告检查
- 沙箱镜像修复
- 沙箱作用域警告

### 6. Hooks 和工作空间
- Gmail Hooks 模型验证
- 工作空间状态检查
- 工作空间备份提示
- 内存系统提示

### 7. CLI 和工具
- Shell 补全检查和修复
- 更新检查和提示
- 源安装问题检查

## 使用示例
```bash
# 运行完整诊断
openclaw doctor

# 非交互式模式
openclaw doctor --non-interactive

# 生成并配置网关令牌
openclaw doctor --generate-gateway-token

# 跳过工作空间建议
openclaw doctor --no-workspace-suggestions

# 应用修复
openclaw doctor --fix
```

## 返回值
成功执行后：
- 0 - 所有检查通过
- 1 - 有警告或问题
- 2 - 严重错误

## 修复操作
诊断可以修复的问题包括：
- 配置格式错误
- 旧版状态迁移
- 沙箱镜像问题
- 网关认证配置
- 网关守护进程状态
- Shell 补全配置