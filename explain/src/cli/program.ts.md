# 文件解释：src/cli/program.ts

## 文件路径
`src/cli/program.ts`

## 文件用途
CLI 程序定义的导出模块，导出构建 CLI 程序的核心函数和端口管理工具。

## 主要类/函数

### 导出内容
- `forceFreePort` - 从 `./ports.js` 导出的强制释放端口函数
- `buildProgram` - 从 `./program/build-program.js` 导出的构建 CLI 程序函数

## 主要依赖

- `./ports.js` - 端口管理
- `./program/build-program.js` - CLI 程序构建器

## 重要逻辑说明

这是一个简单的导出模块，将 CLI 程序构建相关的功能集中导出，方便其他模块引用。`buildProgram` 函数用于创建和配置 Commander.js 程序实例，而 `forceFreePort` 用于端口管理。

## 行数统计
3 行
