# utils/provider-utils.ts 解读

**文件路径**: `src/utils/provider-utils.ts`

## 文件用途
提供商特定的逻辑和功能判断工具，用于处理不同 AI 模型提供商的特殊需求和行为。

## 主要函数

### isReasoningTagProvider(provider: string | undefined | null): boolean
- **功能**: 判断提供商是否需要使用标签包装推理内容
- **参数**: 提供商标识符
- **返回值**: 布尔值
- **实现逻辑**:
  1. 标准化提供商名称（去空白、转小写）
  2. 检查精确匹配或已知前缀
  3. 返回是否需要标签包装

## 提供商推理标签需求

### 需要标签包装的提供商
- `google-gemini-cli` - Google Gemini CLI
- `google-generative-ai` - Google 生成式 AI
- `google-antigravity` 及其模型变体（如 `google-antigravity/gemini-3`）
- `minimax` 及相关提供商（M2.1 是聊天/推理类型）

### 不需要标签包装的提供商
- `ollama` - 其 OpenAI 兼容端点通过 `reasoning` 字段原生支持推理

## 推理标签格式
需要标签包装的提供商会在文本流中使用标签：
- `` - 推理/思考开始
- `<final>` - 最终答案开始
- `` - 推理/思考结束

## 设计原因

### 为什么需要标签包装？
某些提供商没有原生的推理/思考 API 字段，需要通过文本标记来区分推理内容和最终答案。

### 为什么 Ollama 不需要？
Ollama 的 OpenAI 兼容端点在流式响应块中通过 `reasoning` 字段原生支持推理，强制标签包装会导致输出被丢弃为 "(no output)"（问题 #2279）。

## 使用场景
- 推理输出格式化
- 提供商特定行为处理
- 流式响应解析
- 思考链渲染

## 代码行数
37 行

## 重要特性
- 提供商名称标准化
- 支持模型变体（如 `google-antigravity/gemini-3`）
- 明确的排除规则（Ollama）
- 防止输出损坏

## 使用示例
```typescript
// Google Gemini 需要标签包装
const geminiNeedsTags = isReasoningTagProvider("google-gemini-cli");
// true

// Ollama 不需要标签包装
const ollamaNeedsTags = isReasoningTagProvider("ollama");
// false

// Antigravity 模型变体
const antigravityNeedsTags = isReasoningTagProvider("google-antigravity/gemini-3");
// true

// Minimax 需要标签包装
const minimaxNeedsTags = isReasoningTagProvider("minimax");
// true

// 未知提供商
const unknownNeedsTags = isReasoningTagProvider("unknown-provider");
// false
```