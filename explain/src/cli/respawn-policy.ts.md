# 文件解释：src/cli/respawn-policy.ts

## 文件路径
`src/cli/respawn-policy.ts`

## 文件用途
CLI 重启策略判断，决定是否需要重启 CLI 进程。

## 主要类/函数

### `shouldSkipRespawnForArgv(argv: string[]): boolean`
判断是否应该跳过重启。

**逻辑**：当参数包含帮助或版本标志时跳过重启（`hasHelpOrVersion` 检查 `-h`, `--help`, `-v`, `-V`, `--version`）。

## 主要依赖

- `./argv.js` - 参数解析（hasHelpOrVersion）

## 重要逻辑说明

这个文件提供重启策略决策。当用户查询帮助或版本信息时，不需要完整的 CLI 初始化流程（包括可能的重启），因此跳过重启以提高响应速度。

## 行数统计
6 行
