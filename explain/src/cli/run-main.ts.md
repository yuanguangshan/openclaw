# 文件解释：src/cli/run-main.ts

## 文件路径
`src/cli/run-main.ts`

## 文件用途
CLI 主运行器，负责初始化环境、加载配置、注册命令并解析执行。

## 主要类/函数

### `rewriteUpdateFlagArgv(argv: string[]): string[]`
将 `--update` 标志重写为 `update` 命令，支持旧式参数格式。

### `shouldRegisterPrimarySubcommand(argv: string[]): boolean`
判断是否应该注册主子命令（当不是帮助或版本查询时）。

### `shouldSkipPluginCommandRegistration(params): boolean`
判断是否应该跳过插件命令注册。当存在内置主命令或没有主命令且为帮助查询时跳过。

### `shouldEnsureCliPath(argv: string[]): boolean`
判断是否应该确保 CLI 在 PATH 中。对于只读命令（status、health、config get 等）不需要。

### `runCli(argv?: string[]): Promise<void>`
主要的 CLI 运行函数，执行完整的 CLI 启动流程：
1. 标准化参数
2. 加载 .env 文件
3. 标准化环境变量
4. 确保 CLI 在 PATH 中
5. 断言运行时支持
6. 尝试路由 CLI（特殊命令处理）
7. 启用控制台捕获
8. 构建程序
9. 安装全局错误处理器
10. 注册主命令
11. 注册插件命令
12. 解析并执行命令

### `isCliMainModule(): boolean`
检查当前是否为 CLI 主模块（直接运行而非被导入）。

## 主要依赖

- `node:process` - 进程管理
- `node:url` - URL 处理
- `../infra/dotenv.js` - .env 文件加载
- `../infra/env.js` - 环境变量标准化
- `../infra/errors.js` - 错误格式化
- `../infra/is-main.js` - 主模块检测
- `../infra/path-env.js` - PATH 环境变量处理
- `../infra/runtime-guard.js` - 运行时支持断言
- `../infra/unhandled-rejections.js` - 未处理的 Promise 拒绝处理
- `../logging.js` - 日志系统
- `./argv.js` - 参数解析
- `./route.js` - CLI 路由
- `./windows-argv.js` - Windows 参数标准化

## 重要逻辑说明

1. **初始化顺序**：
   - 环境设置（.env、环境变量标准化）
   - PATH 检查
   - 运行时验证
   - 特殊命令路由
   - 日志捕获
   - 命令注册
   - 程序解析

2. **命令注册策略**：
   - 根据主命令名称注册对应的内置命令或子 CLI
   - 只有在必要时才注册插件命令（性能优化）
   - 内置命令优先于插件命令

3. **错误处理**：
   - 安装未处理的 Promise 拒绝处理器
   - 安装未捕获的异常处理器
   - 所有错误都会格式化输出并优雅退出

4. **参数标准化**：将 `--update` 标志转换为 `update` 命令，支持旧的命令格式。

5. **性能优化**：
   - 只读命令跳过 PATH 检查
   - 有内置命令时跳过插件注册
   - 帮助查询时跳过某些初始化

## 行数统计
129 行
