# utils/fetch-timeout.ts 解读

**文件路径**: `src/utils/fetch-timeout.ts`

## 文件用途
为 fetch API 添加超时功能的工具模块，提供可靠的网络请求超时控制。

## 主要函数

### relayAbort(this: AbortController): void
- **功能**: 转发中断信号
- **实现**: 使用 `bind()` 避免闭包作用域捕获（防止内存泄漏）
- **特点**: 不转发 Event 参数作为中断原因

### bindAbortRelay(controller: AbortController): () => void
- **功能**: 为 AbortController 创建绑定的中断中继器
- **参数**: `controller` - 中断控制器实例
- **返回值**: 绑定的中断函数，可作为事件监听器使用

### fetchWithTimeout(url: string, init: RequestInit, timeoutMs: number, fetchFn: typeof fetch = fetch): Promise<Response>
- **功能**: 带超时控制的 fetch 包装器
- **参数**:
  - `url`: 请求的 URL
  - `init`: 请求初始化选项（headers、method、body 等）
  - `timeoutMs`: 超时时间（毫秒）
  - `fetchFn`: fetch 实现函数（默认使用全局 fetch）
- **返回值**: Promise<Response>
- **异常**: 超时时抛出 AbortError
- **实现逻辑**:
  1. 创建 AbortController 实例
  2. 设置定时器，超时后调用 `controller.abort()`
  3. 执行 fetch 请求，传入中断信号
  4. 清理定时器

## 主要特性
1. **内存安全**: 使用 `bind()` 而非闭包，避免内存泄漏
2. **超时控制**: 精确控制请求超时时间
3. **灵活性**: 支持自定义 fetch 实现
4. **资源清理**: 确保定时器被正确清理

## 使用场景
- HTTP 请求超时控制
- API 调用保护
- 防止网络请求无限挂起
- 改善用户体验

## 代码行数
38 行

## 使用示例
```typescript
try {
  const response = await fetchWithTimeout(
    'https://api.example.com/data',
    { method: 'GET' },
    5000  // 5秒超时
  );
  const data = await response.json();
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('请求超时');
  }
}
```

## 设计优势
- 使用 AbortController 标准 API
- 最小化依赖，无第三方库
- 类型安全的 TypeScript 实现
- 适合生产环境使用