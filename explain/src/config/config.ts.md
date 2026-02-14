# 文件解释：src/config/config.ts

## 文件路径
`src/config/config.ts`

## 文件用途
配置模块的导出中心，统一导出配置相关的所有功能。

## 主要类/函数

### 配置 I/O 操作
- `clearConfigCache` - 清除配置缓存
- `createConfigIO` - 创建配置 I/O 对象
- `loadConfig` - 加载配置
- `parseConfigJson5` - 解析 JSON5 格式配置
- `readConfigFileSnapshot` - 读取配置文件快照
- `readConfigFileSnapshotForWrite` - 读取用于写入的配置文件快照
- `resolveConfigSnapshotHash` - 解析配置快照哈希
- `writeConfigFile` - 写入配置文件

### 迁移功能
- `migrateLegacyConfig` - 迁移旧版配置

### 路径和运行时
- 所有从 `./paths.js` 导出的内容
- 所有从 `./runtime-overrides.js` 导出的内容

### 类型定义
- 所有从 `./types.js` 导出的类型

### 验证功能
- `validateConfigObject` - 验证配置对象
- `validateConfigObjectRaw` - 验证原始配置对象
- `validateConfigObjectRawWithPlugins` - 验证包含插件的原始配置对象
- `validateConfigObjectWithPlugins` - 验证包含插件的配置对象

### Schema
- `OpenClawSchema` - OpenClaw Zod 验证 Schema

## 主要依赖

- `./io.js` - 配置 I/O 操作
- `./legacy-migrate.js` - 旧版配置迁移
- `./paths.js` - 配置路径
- `./runtime-overrides.js` - 运行时覆盖
- `./types.js` - 类型定义
- `./validation.js` - 配置验证
- `./zod-schema.js` - Zod Schema

## 重要逻辑说明

这是一个统一的导出模块，将配置系统的各个功能模块集中导出，便于其他模块引用。这种设计遵循关注点分离原则，每个子模块负责特定功能，通过此模块提供统一的访问接口。

## 行数统计
22 行
