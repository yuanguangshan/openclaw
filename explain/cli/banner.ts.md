# cli/banner.ts 解读

**文件路径**: `src/cli/banner.ts`

## 文件用途
CLI 横幅和品牌显示模块，负责生成 OpenClaw 命令行界面的欢迎信息和 ASCII 艺术。

## 主要类型定义

### BannerOptions
横幅选项接口：
- `argv`: 命令行参数数组
- `commit`: Git 提交哈希
- `columns`: 终端列数
- `richTty`: 富文本 TTY 模式
- 继承自 TaglineOptions

## 主要常量

### LOBSTER_ASCII
龙虾 ASCII 艺术数组，用于显示 OpenClaw 的品牌形象。

## 主要函数

### splitGraphemes(value: string): string[]
- **功能**: 按字形（grapheme）分割字符串
- **参数**: 输入字符串
- **返回值**: 字形数组
- **实现**:
  - 使用 `Intl.Segmenter`（如果可用）
  - 回退到字符分割

### formatCliBannerLine(version: string, options: BannerOptions = {}): string
- **功能**: 格式化 CLI 横幅行
- **参数**:
  - `version`: 版本字符串
  - `options`: 横幅选项
- **返回值**: 格式化的横幅字符串
- **实现逻辑**:
  1. 获取提交哈希和标语
  2. 检查终端宽度和富文本支持
  3. 根据终端宽度决定单行或双行显示
  4. 应用主题颜色（富文本模式）

### formatCliBannerArt(options: BannerOptions = {}): string
- **功能**: 格式化 ASCII 艺术横幅
- **参数**: 横幅选项
- **返回值**: 格式化的 ASCII 艺术
- **实现逻辑**:
  1. 纯文本模式：返回原始 ASCII
  2. 富文本模式：为不同字符应用主题颜色
     - `█`: 亮强调色
     - `░`: 暗强调色
     - `▀`: 强调色
     - 其他: 静音色

### emitCliBanner(version: string, options: BannerOptions = {})
- **功能**: 输出 CLI 横幅到标准输出
- **参数**:
  - `version`: 版本字符串
  - `options`: 横幅选项
- **显示条件**:
  - 尚未输出过横幅
  - TTY 终端
  - 不是 JSON 输出
  - 不是版本查询

### hasEmittedCliBanner(): boolean
- **功能**: 检查是否已输出横幅
- **返回值**: 布尔值

## 辅助函数

### hasJsonFlag(argv: string[])
检查是否启用 JSON 输出模式

### hasVersionFlag(argv: string[])
检查是否查询版本信息

## 主要依赖
- `../infra/git-commit.js` - Git 提交哈希解析
- `../terminal/ansi.js` - ANSI 颜色处理
- `../terminal/theme.js` - 终端主题
- `./tagline.js` - 标语选择

## 使用场景
- CLI 启动时显示品牌信息
- 版本信息展示
- 终端美观显示
- 品牌识别

## 代码行数
133 行

## 重要特性
- 响应式布局（根据终端宽度调整）
- 富文本和纯文本双模式
- Unicode 字形支持
- 防止重复输出
- 条件性显示（TTY、JSON、版本）

## 显示效果示例

### 单行模式（宽终端）
```
🦞 OpenClaw 2025.02.15 (abc1234) — Your AI Assistant
```

### 双行模式（窄终端）
```
🦞 OpenClaw 2025.02.15 (abc1234)
     Your AI Assistant
```

### ASCII 艺术
```
▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
██░▄▄▄░██░▄▄░██░▄▄▄██░▀██░██░▄▄▀██░████░▄▄▀██░███░██
██░███░██░▀▀░██░▄▄▄██░█░█░██░█████░████░▀▀░██░█░█░██
██░▀▀▀░██░█████░▀▀▀██░██▄░██░▀▀▄██░▀▀░█░██░██▄▀▄▀▄██
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
                  🦞 OPENCLAW 🦞
```