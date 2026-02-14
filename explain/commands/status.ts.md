# commands/status.ts 解读

**文件路径**: `src/commands/status.ts`

## 文件用途
状态命令模块的导出文件，统一导出所有状态相关的命令和类型。

## 导出内容

### 命令
- `statusCommand` - 主要的状态检查命令

### 函数
- `getStatusSummary` - 获取状态摘要

### 类型
- `SessionStatus` - 会话状态类型
- `StatusSummary` - 状态摘要类型

## 主要依赖
- `./status.command.js` - 状态命令实现
- `./status.summary.js` - 状态摘要生成
- `./status.types.js` - 类型定义

## 使用场景
- CLI 状态检查
- 系统健康状态报告
- 会话状态查询
- 配置验证

## 代码行数
4 行

## 重要特性
- 统一导出接口
- 模块化组织
- 类型安全

## 说明
这是一个导出文件，主要功能由其他模块实现，此文件用于统一导出所有状态相关的功能。