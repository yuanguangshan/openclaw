# commands/configure.wizard.ts 解读

**文件路径**: `src/commands/configure.wizard.ts`

## 文件用途
配置向导（Wizard）主入口，提供交互式的配置流程。

## 主要函数

### promptConfigureSection(runtime): Promise<ConfigureSectionChoice>
- **功能**: 提示用户选择配置区块
- **参数**: 运行时环境
- **返回值**: 选择的配置区块
- **实现逻辑**:
  1. 显示选择列表（继续、跳过）
  2. 等待用户选择

### runConfigureWizard(opts, runtime)
- **功能**: 运行配置向导
- **参数**:
  - `opts`: 配置选项
  - `runtime`: 运行时环境
- **实现逻辑**:
  1. 初始化 Prompter
  2. 加载现有配置
  3. 根据用户选择执行配置区块
  4. 验证配置完整性
  5. 写入配置文件
  6. 显示完成信息

## 主要依赖
- `../config/config.js` - 配置类型
- `../runtime.js` - 运行时类型
- `./configure.commands.js` - 配置命令
- `./configure.gateway-auth.js` - 网关认证配置
- `./configure.shared.js` - 共享配置工具

## 使用场景
- 首次配置 OpenClaw
- 修改现有配置
- 修复配置问题
- 更新网关配置
- 配置认证凭据

## 代码行数
595 行

## 重要特性
- 交互式向导流程
- 多区块配置支持
- 配置验证和修复
- 完善的错误提示
- 用户友好的体验

## 配置区块类型 (CONFIGURE_WIZARD_SECTIONS)
根据配置的复杂程度，将配置分为多个逻辑区块。

## 使用示例
```bash
# 运行完整配置向导
openclaw configure

# 查看配置向导
openclaw configure --help
```