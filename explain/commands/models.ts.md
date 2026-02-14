# commands/models.ts 解读

**文件路径**: `src/commands/models.ts`

## 文件用途
模型管理命令模块的导出文件，统一导出所有模型相关的命令和功能。

## 导出内容

### GitHub Copilot 认证
- `githubCopilotLoginCommand` - GitHub Copilot 登录命令

### 模型别名管理
- `modelsAliasesAddCommand` - 添加模型别名
- `modelsAliasesListCommand` - 列出模型别名
- `modelsAliasesRemoveCommand` - 移除模型别名

### 模型认证管理
- `modelsAuthAddCommand` - 添加模型认证
- `modelsAuthLoginCommand` - 模型登录
- `modelsAuthPasteTokenCommand` - 粘贴令牌
- `modelsAuthSetupTokenCommand` - 设置令牌

### 模型认证顺序管理
- `modelsAuthOrderClearCommand` - 清除认证顺序
- `modelsAuthOrderGetCommand` - 获取认证顺序
- `modelsAuthOrderSetCommand` - 设置认证顺序

### 模型回退管理
- `modelsFallbacksAddCommand` - 添加回退模型
- `modelsFallbacksClearCommand` - 清除回退模型
- `modelsFallbacksListCommand` - 列出回退模型
- `modelsFallbacksRemoveCommand` - 移除回退模型

### 图像模型回退管理
- `modelsImageFallbacksAddCommand` - 添加图像回退模型
- `modelsImageFallbacksClearCommand` - 清除图像回退模型
- `modelsImageFallbacksListCommand` - 列出图像回退模型
- `modelsImageFallbacksRemoveCommand` - 移除图像回退模型

### 模型列表和扫描
- `modelsListCommand` - 列出模型
- `modelsStatusCommand` - 模型状态
- `modelsScanCommand` - 扫描模型
- `modelsSetCommand` - 设置模型
- `modelsSetImageCommand` - 设置图像模型

## 主要依赖
- `../providers/github-copilot-auth.js` - GitHub Copilot 认证
- `./models/aliases.js` - 别名管理
- `./models/auth.js` - 认证管理
- `./models/auth-order.js` - 认证顺序
- `./models/fallbacks.js` - 回退模型
- `./models/image-fallbacks.js` - 图像回退
- `./models/list.js` - 模型列表
- `./models/scan.js` - 模型扫描
- `./models/set.js` - 模型设置
- `./models/set-image.js` - 图像模型设置

## 使用场景
- 模型配置管理
- 多提供商认证
- 模型别名设置
- 回退策略配置
- 模型状态查询

## 代码行数
34 行

## 重要特性
- 统一的命令导出
- 模块化组织
- 多提供商支持
- 完整的模型生命周期管理

## 说明
这是一个导出文件，主要功能由各个子模块实现，此文件用于统一导出所有模型相关的命令和功能。

## 使用示例
```bash
# 列出所有模型
openclaw models list

# 添加模型别名
openclaw models aliases add g4 gpt-4

# 设置模型
openclaw models set gpt-4

# 配置认证
openclaw models auth login anthropic
```