# 文件解释：src/cli/progress.ts

## 文件路径
`src/cli/progress.ts`

## 文件用途
CLI 进度显示系统，支持多种进度显示模式（OSC 进度、Spinner、单行、日志），并自动选择最佳的显示方式。

## 主要类/函数

### `createCliProgress(options: ProgressOptions): ProgressReporter`
创建进度报告器，根据终端能力自动选择最佳显示方式。

**ProgressOptions 配置**：
- `label`: 进度标签
- `indeterminate`: 是否为不确定进度
- `total`: 总数（用于确定进度）
- `enabled`: 是否启用
- `delayMs`: 延迟显示时间（毫秒）
- `stream`: 输出流（默认 stderr）
- `fallback`: 回退模式（"spinner" | "line" | "log" | "none"）

**ProgressReporter 接口**：
- `setLabel(label)`: 更新标签
- `setPercent(percent)`: 设置百分比（0-100）
- `tick(delta?)`: 增加进度
- `done()`: 完成进度

### `withProgress<T>(options, work): Promise<T>`
进度包装器，自动管理进度生命周期。执行工作函数并在完成后清理进度。

### `withProgressTotals<T>(options, work): Promise<T>`
支持总数更新的进度包装器。工作函数接收一个更新回调，可以传递 `completed` 和 `total` 来更新进度。

## 类型定义

```typescript
type ProgressReporter = {
  setLabel: (label: string) => void;
  setPercent: (percent: number) => void;
  tick: (delta?: number) => void;
  done: () => void;
};

type ProgressTotalsUpdate = {
  completed: number;
  total: number;
  label?: string;
};
```

## 主要依赖

- `@clack/prompts` - CLI 提示库（用于 spinner）
- `osc-progress` - OSC 进度协议（支持终端进度条）
- `../terminal/progress-line.js` - 进度行管理
- `../terminal/theme.js` - 终端主题

## 重要逻辑说明

1. **模式选择优先级**：
   - OSC 进度：最佳体验，支持现代终端的集成进度条
   - Spinner：备选，使用 Clack 的 spinner
   - 单行模式：简单的文本行显示
   - 日志模式：非 TTY 环境下使用，输出日志行

2. **自动适配**：
   - 检测终端是否为 TTY
   - 检测是否支持 OSC 进度协议
   - 根据 fallback 选项选择显示方式

3. **并发控制**：使用 `activeProgress` 计数器防止多个进度同时显示（嵌套进度会被忽略）。

4. **延迟启动**：支持延迟启动进度显示，避免快速操作时闪烁。

5. **节流**：日志模式下使用 250ms 节流，避免输出过于频繁。

6. **状态管理**：
   - `indeterminate` 模式：不显示百分比
   - 确定模式：显示百分比和进度条
   - 自动切换：第一次调用 setPercent 会切换到确定模式

## 行数统计
231 行
