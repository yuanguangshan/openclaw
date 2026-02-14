# shared/node-match.ts 解读

**文件路径**: `src/shared/node-match.ts`

## 文件用途
节点匹配和解析工具模块，用于通过查询字符串匹配和解析 OpenClaw 节点。

## 主要类型定义

### NodeMatchCandidate
节点匹配候选对象：
- `nodeId`: 节点 ID
- `displayName`: 显示名称
- `remoteIp`: 远程 IP 地址

## 主要函数

### normalizeNodeKey(value: string): string
- **功能**: 标准化节点键
- **参数**: 节点标识符字符串
- **返回值**: 标准化后的节点键
- **实现逻辑**:
  - 转换为小写
  - 非字母数字字符替换为连字符
  - 去除首尾连字符

### listKnownNodes(nodes: NodeMatchCandidate[]): string
- **功能**: 列出已知节点
- **参数**: 节点候选数组
- **返回值**: 逗号分隔的节点列表字符串
- **实现逻辑**: 使用显示名称或远程 IP 或节点 ID

### resolveNodeMatches(nodes: NodeMatchCandidate[], query: string): NodeMatchCandidate[]
- **功能**: 根据查询解析匹配的节点
- **参数**:
  - `nodes`: 节点候选数组
  - `query`: 查询字符串
- **返回值**: 匹配的节点数组
- **匹配规则**:
  1. 精确匹配节点 ID
  2. 精确匹配远程 IP
  3. 标准化后的显示名称匹配
  4. 节点 ID 前缀匹配（查询长度 ≥ 6）

### resolveNodeIdFromCandidates(nodes: NodeMatchCandidate[], query: string): string
- **功能**: 从候选节点中解析出唯一的节点 ID
- **参数**:
  - `nodes`: 节点候选数组
  - `query`: 查询字符串
- **返回值**: 解析出的节点 ID
- **异常处理**:
  - 空查询 → 抛出 "node required" 错误
  - 无匹配 → 抛出 "unknown node" 错误
  - 多个匹配 → 抛出 "ambiguous node" 错误

## 使用场景
- 节点选择和路由
- 用户输入节点标识符解析
- 节点管理和查找
- 分布式节点发现

## 代码行数
70 行

## 重要特性
- 灵活的匹配策略
- 清晰的错误提示
- 支持多种标识方式
- 智能前缀匹配

## 使用示例
```typescript
const nodes = [
  { nodeId: "node-12345", displayName: "Production Server" },
  { nodeId: "node-67890", displayName: "Dev Server" },
  { nodeId: "node-abcde", remoteIp: "192.168.1.100" }
];

// 精确匹配
const matches = resolveNodeMatches(nodes, "node-12345");
// [{ nodeId: "node-12345", displayName: "Production Server" }]

// 前缀匹配
const prefixMatches = resolveNodeMatches(nodes, "node-123");
// [{ nodeId: "node-12345", displayName: "Production Server" }]

// 唯一解析
const nodeId = resolveNodeIdFromCandidates(nodes, "node-12345");
// "node-12345"

// 错误情况
try {
  resolveNodeIdFromCandidates(nodes, "node-1");
} catch (error) {
  // Error: ambiguous node: node-1 (matches: node-12345, node-67890)
}
```