# 文件解释：src/cli/prompt.ts

## 文件路径
`src/cli/prompt.ts`

## 文件用途
简单的命令行提示工具，用于获取用户确认（是/否）。

## 主要类/函数

### `promptYesNo(question: string, defaultYes?: boolean): Promise<boolean>`
显示一个简单的 Y/N 提示并等待用户输入。

**行为**：
- 如果 `--verbose` 和 `--yes` 都设置了，自动返回 `true`
- 如果设置了 `--yes`，自动返回 `true`
- 否则显示提示并等待用户输入
- 空输入返回默认值
- 以 'y' 开头的输入返回 `true`
- 其他输入返回 `false`

**参数**：
- `question`: 提示问题文本
- `defaultYes`: 默认是否为 "yes"（影响提示后缀）

## 主要依赖

- `node:process` - stdin/stdout
- `node:readline/promises` - readline 接口
- `../globals.js` - 全局标志（isVerbose, isYes）

## 重要逻辑说明

1. **全局标志优先**：全局 `--yes` 标志可以绕过所有提示，实现自动化脚本。

2. **提示后缀**：
   - `defaultYes = true`: 显示 `[Y/n]`（默认 yes）
   - `defaultYes = false`: 显示 `[y/N]`（默认 no）

3. **输入处理**：
   - 自动去除首尾空格
   - 不区分大小写
   - 只要以 'y' 开头就接受为 yes

## 行数统计
22 行
