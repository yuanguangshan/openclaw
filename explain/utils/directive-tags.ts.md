# utils/directive-tags.ts 解读

**文件路径**: `src/utils/directive-tags.ts`

## 文件用途
解析消息文本中的内联指令标签，用于处理音频和回复相关的特殊格式。

## 主要类型定义

### InlineDirectiveParseResult
内联指令解析结果接口：
- `text`: 处理后的文本内容
- `audioAsVoice`: 是否标记为语音模式
- `replyToId`: 回复的消息 ID
- `replyToExplicitId`: 显式指定的回复 ID
- `replyToCurrent`: 是否回复当前消息
- `hasAudioTag`: 是否包含音频标签
- `hasReplyTag`: 是否包含回复标签

### InlineDirectiveParseOptions
内联指令解析选项接口：
- `currentMessageId`: 当前消息 ID
- `stripAudioTag`: 是否移除音频标签（默认 true）
- `stripReplyTags`: 是否移除回复标签（默认 true）

## 主要正则表达式

### AUDIO_TAG_RE
- 模式: `/\[\[\s*audio_as_voice\s*\]\]/gi`
- 用途: 匹配音频语音标签
- 示例: `[[audio_as_voice]]`

### REPLY_TAG_RE
- 模式: `/\[\[\s*(?:reply_to_current|reply_to\s*:\s*([^\]\n]+))\s*\]\]/gi`
- 用途: 匹配回复标签
- 示例: `[[reply_to_current]]` 或 `[[reply_to: msg123]]`

## 主要函数

### normalizeDirectiveWhitespace(text: string): string
- **功能**: 标准化指令中的空白字符
- **参数**: 原始文本
- **返回值**: 标准化后的文本
- **实现逻辑**:
  - 将多个空格/制表符替换为单个空格
  - 规范化换行符周围的空白
  - 去除首尾空白

### parseInlineDirectives(text?: string, options: InlineDirectiveParseOptions = {}): InlineDirectiveParseResult
- **功能**: 解析文本中的内联指令
- **参数**:
  - `text`: 待解析的文本
  - `options`: 解析选项
- **返回值**: 解析结果对象
- **实现逻辑**:
  1. 处理音频标签，设置语音模式标志
  2. 处理回复标签，提取回复目标 ID
  3. 标准化空白字符
  4. 组装解析结果

## 使用场景
- 消息文本预处理
- AI 响应格式控制
- 音频消息识别
- 对话线程管理

## 代码行数
83 行

## 重要特性
- 支持大小写不敏感的标签匹配
- 可配置是否移除标签
- 处理多种回复格式
- 保留原始标签信息用于状态检查

## 标签格式示例
```
正常消息 [[audio_as_voice]]
回复指定消息 [[reply_to: message123]]
回复当前消息 [[reply_to_current]]
```