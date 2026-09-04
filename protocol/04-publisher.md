# HCPP Protocol — Publisher

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Publisher 表示发布数字内容的主体，可以是组织、个人、系统或其他实体。

Publisher 是 HCPP 信任模型的核心之一：密码学只能证明“某条记录由某个私钥签署”，而不能自动证明“这个 Publisher 值得信任”。组织层面的信任需要额外机制（未来可结合 DID / Verifiable Credentials 等）。

---

## 2. 最小数据模型

一个 Publisher 在 v0.1 至少需要：

```json
{
  "id": "hxx:publisher:hxxnewsletter",
  "name": "HxxNewsletter",
  "publicKey": {
    "algorithm": "Ed25519",
    "keyId": "hxx:key:publisher:01",
    "value": "..."
  },
  "status": "active",
  "createdAt": "2026-09-04T08:00:00Z"
}
```

### 字段说明

| 字段 | 要求 | 说明 |
|------|------|------|
| `id` | 必须 | Publisher 唯一标识 |
| `name` | 必须 | 人类可读名称 |
| `publicKey` | 必须 | 当前用于验证的公钥信息 |
| `status` | 必须 | `active` / `revoked` / `suspended` |
| `createdAt` | 推荐 | 创建时间（RFC 3339） |
| `updatedAt` | 可选 | 最后更新时间 |

---

## 3. Publisher ID

v0.1 不强制使用 DID。

推荐形式：

```text
hxx:publisher:{slug}
```

示例：

```text
hxx:publisher:hxxnewsletter
hxx:publisher:example-org
```

未来可平滑增加：

- `did:web:...`
- `did:key:...`
- 其他 DID Method

并保持协议兼容。

---

## 4. 公钥与密钥标识

### 4.1 publicKey 结构

```json
{
  "algorithm": "Ed25519",
  "keyId": "hxx:key:publisher:01",
  "value": "BASE64URL_OR_OTHER_ENCODING"
}
```

- `algorithm`：v0.1 推荐 `Ed25519`
- `keyId`：密钥的唯一标识，用于在签名中引用
- `value`：公钥编码（实现应明确并文档化所用编码）

### 4.2 多密钥与历史密钥

Publisher 可能随时间轮换密钥。实现 MUST 支持：

- 记录当前生效密钥
- 保留历史密钥及其有效期 / 状态
- 使用历史密钥签署的旧记录在密钥轮换后仍能被正确验证

推荐内部维护密钥列表，而不是只保存单一公钥。

---

## 5. 密钥生命周期

### 5.1 生成与存储

- Private Key **MUST NOT** 上传至 Registry 或任何公共日志。
- 推荐使用操作系统密钥存储、硬件安全模块（HSM）或云 KMS。
- SDK 只负责使用调用方提供的 Signer 接口，不托管私钥。

### 5.2 轮换（Key Rotation）

推荐流程：

1. 生成新密钥对，分配新的 `keyId`。
2. 将新公钥注册到 Publisher 记录，标记为当前生效。
3. 旧密钥标记为 `retired` 或设置 `notAfter`。
4. 之后的新记录使用新密钥签名。
5. 历史记录继续用对应历史密钥验证。

### 5.3 撤销（Key Revocation）

- 密钥被泄露或不再使用时，应将其标记为 `revoked`。
- 撤销密钥**不等于**撤销已用该密钥签署的历史内容。
- 验证器在验证旧记录时，仍应能确认“签名在密码学上有效”，同时可提示“签署密钥当前已被撤销”。

---

## 6. Publisher 状态

| 状态 | 含义 |
|------|------|
| `active` | 正常可用 |
| `suspended` | 临时停用（可恢复） |
| `revoked` | 永久撤销 |

状态影响的是**新记录的接受**，而不是历史记录的密码学有效性。

---

## 7. 与 Provenance Record 的关系

Provenance Record 中通过以下方式引用 Publisher：

```json
{
  "publisher": {
    "id": "hxx:publisher:hxxnewsletter"
  }
}
```

签名中的 `keyId` 必须能解析到该 Publisher 下的某个已知公钥。

---

## 8. 未来扩展方向

v0.1 之后可考虑：

- 将 Publisher 与 DID 绑定
- 使用 W3C Verifiable Credentials 表达 Publisher 资质或认证
- 建立简单的 Trust Registry（组织信任层）
- 支持多管理员 / 多签名策略

这些扩展应尽量以向后兼容的方式增加字段，而不是破坏现有结构。

---

## 9. 安全要求摘要

- Private Key 永不离开控制方。
- 支持密钥轮换与撤销，且不影响历史验证。
- Registry 只存储公钥与元数据。
- 验证时同时检查签名有效性与 Publisher / 密钥当前状态。

---

## 10. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
