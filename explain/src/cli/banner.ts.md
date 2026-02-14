# 文件解释：src/cli/banner.ts

## 文件路径
`src/cli/banner.ts`

## 文件用途
CLI 横幅显示功能，包括 ASCII 艺术字和版本信息展示，支持富文本和普通文本两种模式。

## 主要类/函数

### `formatCliBannerLine(version: string, options?: BannerOptions): string`
格式化 CLI 横幅行，包含版本号、提交哈希和标语。支持：
- 单行/双行布局（根据终端宽度自动选择）
- 富文本/纯文本模式
- Git 提交哈希显示

### `formatCliBannerArt(options?: BannerOptions): string`
格式化 CLI 横幅 ASCII 艺术，使用 🦞龙虾 ASCII 字符。在富文本模式下，会对不同字符应用不同的颜色主题。

### `emitCliBanner(version: string, options?: BannerOptions): void`
在终端输出 CLI 横幅。只在以下条件都满足时输出：
- 尚未输出过横幅（通过 bannerEmitted 标志控制）
- stdout 是 TTY 终端
- 没有 `--json` 或 `--version`/`-v`/`-V` 标志

### `hasEmittedCliBanner(): boolean`
检查是否已输出过 CLI 横幅。

### `formatBannerLine` 辅助函数

#### `splitGraphemes(value: string): string[]`
使用 Intl.Segmenter 将字符串分割为 Unicode 字素簇（正确处理 emoji 和复合字符）。

## 类型定义

```typescript
type BannerOptions = TaglineOptions & {
  argv?: string[];
  commit?: string | null;
  columns?: number;
  richTty?: boolean;
};
```

## 主要依赖

- `../infra/git-commit.js` - Git 提交哈希解析
- `../terminal/ansi.js` - ANSI 宽度计算
- `../terminal/theme.js` - 终端主题颜色
- `./tagline.js` - 标语选择

## 重要逻辑说明

1. **横幅内容**：使用 🦞 龙虾 ASCII 艺术（LOBSTER_ASCII 数组），包含 5 行艺术字和 1 行 "OPENCLAW" 文字。

2. **宽度自适应**：根据终端列数（columns）决定是否使用单行或双行布局，确保内容不会超出屏幕宽度。

3. **颜色渲染**：
   - `█` 使用 accentBright 颜色
   - `░` 使用 accentDim 颜色
   - `▀` 使用 accent 颜色
   - 其他字符使用 muted 颜色
   - OPENCLAW 标题使用 accent 和 info 颜色突出显示

4. **输出条件控制**：通过多个条件判断横幅是否应该输出，避免在不适合的场景（如 JSON 输出、版本查询、非 TTY）下显示横幅。

5. **字素处理**：使用 Intl.Segmenter API 正确处理 emoji 和其他复合字符的宽度计算。

## 行数统计
133 行
