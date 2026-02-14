# infra/device-identity.ts 解读

**文件路径**: `src/infra/device-identity.ts`

## 文件用途
设备身份管理模块，负责生成、存储、加载和验证设备身份（ED25519 密钥对），用于设备认证和签名。

## 主要类型定义

### DeviceIdentity
设备身份对象
- `deviceId: string` - 设备 ID（公钥的 SHA-256 指纹）
- `publicKeyPem: string` - 公钥（PEM 格式）
- `privateKeyPem: string` - 私钥（PEM 格式）

### StoredIdentity
存储的设备身份
- `version: 1` - 版本号
- `deviceId: string` - 设备 ID
- `publicKeyPem: string` - 公钥
- `privateKeyPem: string` - 私钥
- `createdAtMs: number` - 创建时间戳

## 主要函数

### loadOrCreateDeviceIdentity(filePath?): DeviceIdentity
- **功能**: 加载或创建设备身份
- **参数**: 身份文件路径（默认：`<stateDir>/identity/device.json`）
- **返回值**: 设备身份对象
- **实现逻辑**:
  1. 尝试从文件加载现有身份
  2. 验证文件格式和版本
  3. 重新计算设备 ID（指纹）并验证一致性
  4. 如果不匹配，更新文件
  5. 如果加载失败，生成新身份
  6. 确保文件权限为 0o600
  7. 返回设备身份

### signDevicePayload(privateKeyPem, payload): string
- **功能**: 使用设备私钥签名数据
- **参数**:
  - `privateKeyPem` - 私钥（PEM 格式）
  - `payload` - 要签名的数据（字符串）
- **返回值**: Base64URL 编码的签名
- **实现逻辑**:
  1. 从 PEM 创建私钥对象
  2. 对数据进行 ED25519 签名
  3. 返回 Base64URL 编码的签名

### normalizeDevicePublicKeyBase64Url(publicKey): string | null
- **功能**: 规范化设备公钥为 Base64URL 格式
- **参数**: 公钥（PEM 或 Base64URL 格式）
- **返回值**: Base64URL 格式的公钥或 null（失败）
- **实现逻辑**:
  1. 检测输入格式（PEM 或 Base64URL）
  2. 提取原始公钥字节
  3. 转换为 Base64URL 格式
  4. 失败时返回 null

### deriveDeviceIdFromPublicKey(publicKey): string | null
- **功能**: 从公钥推导设备 ID
- **参数**: 公钥（PEM 或 Base64URL 格式）
- **返回值**: 设备 ID（SHA-256 十六进制）或 null（失败）
- **实现逻辑**:
  1. 提取原始公钥字节
  2. 计算 SHA-256 哈希
  3. 返回十六进制字符串

### publicKeyRawBase64UrlFromPem(publicKeyPem): string
- **功能**: 从 PEM 公钥提取原始字节并编码为 Base64URL
- **参数**: PEM 格式的公钥
- **返回值**: Base64URL 编码的原始公钥
- **实现逻辑**:
  1. 提取原始公钥字节
  2. 转换为 Base64URL 格式

### verifyDeviceSignature(publicKey, payload, signatureBase64Url): boolean
- **功能**: 验证设备签名
- **参数**:
  - `publicKey` - 公钥（PEM 或 Base64URL 格式）
  - `payload` - 原始数据
  - `signatureBase64Url` - 签名（Base64URL 格式）
- **返回值**: 签名是否有效
- **实现逻辑**:
  1. 解析公钥格式
  2. 创建公钥对象
  3. 解码签名
  4. 验证签名
  5. 失败时返回 false

## 辅助函数

### derivePublicKeyRaw(publicKeyPem): Buffer
- **功能**: 从 PEM 公钥提取原始字节
- **参数**: PEM 格式的公钥
- **返回值**: 原始公钥字节
- **实现逻辑**:
  1. 创建公钥对象
  2. 导出为 SPKI DER 格式
  3. 对于 ED25519，移除前缀返回原始 32 字节
  4. 其他情况返回完整 DER

### fingerprintPublicKey(publicKeyPem): string
- **功能**: 生成公钥指纹（设备 ID）
- **参数**: PEM 格式的公钥
- **返回值**: SHA-256 十六进制字符串
- **实现逻辑**:
  1. 提取原始公钥字节
  2. 计算 SHA-256 哈希
  3. 返回十六进制字符串

### generateIdentity(): DeviceIdentity
- **功能**: 生成新的设备身份
- **参数**: 无
- **返回值**: 新的设备身份对象
- **实现逻辑**:
  1. 生成 ED25519 密钥对
  2. 导出 PEM 格式公钥和私钥
  3. 计算设备 ID（公钥指纹）
  4. 返回身份对象

### base64UrlEncode(buf): string
- **功能**: Base64URL 编码
- **参数**: Buffer
- **返回值**: Base64URL 字符串
- **实现逻辑**:
  1. 标准 Base64 编码
  2. 替换 `+` 为 `-`
  3. 替换 `/` 为 `_`
  4. 移除尾部 `=`

### base64UrlDecode(input): Buffer
- **功能**: Base64URL 解码
- **参数**: Base64URL 字符串
- **返回值**: Buffer
- **实现逻辑**:
  1. 替换 `-` 为 `+`
  2. 替换 `_` 为 `/`
  3. 添加必要的 `=` 填充
  4. 标准 Base64 解码

## 主要依赖
- `node:crypto` - 加密操作（密钥生成、签名、验证）
- `node:fs` - 文件系统操作
- `node:path` - 路径处理
- `../config/paths.js` - 状态目录路径

## 使用场景
- 设备认证和授权
- 设备配对和安全通信
- API 请求签名和验证
- 防止设备身份伪造
- 设备唯一标识

## 代码行数
183 行

## 重要特性
- ED25519 签名算法（安全且高效）
- 自动密钥对生成
- 持久化存储（文件）
- 安全的文件权限（0o600）
- 自动修复设备 ID 不一致
- 灵活的公钥格式支持（PEM、Base64URL）
- Base64URL 编码（URL 友好）

## 默认行为
- **身份文件**: `<stateDir>/identity/device.json`
- **文件权限**: 0o600（仅所有者读写）
- **算法**: ED25519
- **设备 ID**: 公钥的 SHA-256 指纹

## 存储格式

### JSON 结构
```json
{
  "version": 1,
  "deviceId": "a1b2c3d4e5f6...",
  "publicKeyPem": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
  "privateKeyPem": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
  "createdAtMs": 1739638800000
}
```

## ED25519 特性
- **密钥长度**: 32 字节
- **签名长度**: 64 字节
- **安全性**: 高强度签名算法
- **性能**: 快速签名和验证

## 使用示例

### 加载或创建设备身份
```typescript
const identity = loadOrCreateDeviceIdentity();
// {
//   deviceId: "a1b2c3d4e5f6...",
//   publicKeyPem: "-----BEGIN PUBLIC KEY-----...",
//   privateKeyPem: "-----BEGIN PRIVATE KEY-----...",
// }

// 自定义路径
const identity = loadOrCreateDeviceIdentity("/custom/path/identity.json");
```

### 签名数据
```typescript
const identity = loadOrCreateDeviceIdentity();
const payload = "Hello, World!";
const signature = signDevicePayload(identity.privateKeyPem, payload);
// "x7s8d9f0g1h2..."
```

### 验证签名
```typescript
const identity = loadOrCreateDeviceIdentity();
const payload = "Hello, World!";
const signature = signDevicePayload(identity.privateKeyPem, payload);

const isValid = verifyDeviceSignature(
  identity.publicKeyPem,
  payload,
  signature
);
// true
```

### 规范化公钥
```typescript
// 从 PEM 规范化
const normalized = normalizeDevicePublicKeyBase64Url(identity.publicKeyPem);
// "yZxYwXvUtsrqp..."

// 从 Base64URL 验证
const id = deriveDeviceIdFromPublicKey(normalized);
// "a1b2c3d4e5f6..."
```

### 提取原始公钥
```typescript
const rawKey = publicKeyRawBase64UrlFromPem(identity.publicKeyPem);
// "yZxYwXvUtsrqp..."
```

## 安全考虑

### 文件权限
- 身份文件权限设置为 0o600
- 仅所有者可读写
- 其他用户无法访问

### 密钥管理
- 私钥从不离开内存（除非存储）
- 文件权限保护防止未授权访问
- ED25519 提供强签名保证

### 验证
- 签名验证失败返回 false（不抛出异常）
- 防止基于异常的信息泄露

## 错误处理

### 加载失败
- 如果文件损坏或格式错误，自动生成新身份
- 所有异常被捕获并处理

### 签名验证
- 任何验证失败返回 false
- 不会抛出异常

### 公钥规范化
- 无效输入返回 null
- 不会抛出异常

## Base64URL 编码

### 为什么使用 Base64URL
- URL 安全（无特殊字符）
- 无需 URL 编码
- 与 JWT 和其他标准兼容

### 转换规则
- `+` → `-`
- `/` → `_`
- 移除尾部 `=`

## 设备 ID 计算

### 算法
1. 提取原始公钥字节（ED25519 为 32 字节）
2. 计算 SHA-256 哈希
3. 返回 64 个十六进制字符

### 稳定性
- 相同公钥始终产生相同设备 ID
- 不受时间或位置影响
- 可用于唯一标识设备

## 兼容性

### 公钥格式
- **PEM 格式**: `-----BEGIN PUBLIC KEY-----...`
- **Base64URL**: 原始字节的 URL 安全编码
- **自动检测**: 函数自动检测输入格式

### 签名格式
- **Base64URL**: 默认格式
- **Base64**: 备用格式（验证时支持）

## 迁移和修复

### 设备 ID 修复
如果存储的 `deviceId` 与公钥不匹配：
1. 重新计算正确的设备 ID
2. 更新文件中的 `deviceId` 字段
3. 保持密钥不变

这确保了向后兼容性和数据一致性。
