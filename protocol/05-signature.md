# HCPP Protocol — Digital Signature

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

数字签名用于证明某条 Provenance Record 由特定 Publisher 的私钥签署，且签署后内容未被篡改。

HCPP v0.1 推荐使用 **Ed25519**。

签名的对象是**规范化后的 Provenance Record**，而不是原始文章 HTML 或渲染结果。

---

## 2. 推荐算法

| 项目 | v0.1 推荐 |
|------|-----------|
| 签名算法 | Ed25519 |
| 规范化 | RFC 8785 JCS |
| 编码 | 签名值推荐使用 Base64URL（无填充） |

未来可通过 `algorithm` 字段扩展其他算法，验证器根据声明选择对应验证逻辑。

---

## 3. 签名流程

```text
构造待签名的 Provenance Record
          ↓
去除或清空 signature 字段
          ↓
JCS Canonicalization（RFC 8785）
          ↓
得到确定性 UTF-8 字节序列
          ↓
Ed25519 Sign（使用 Publisher 私钥）
          ↓
将签名结果写入 signature 字段
```

### 3.1 待签名内容

- 参与签名的 JSON 对象在进行 JCS 之前，**不应包含有效的 signature 值**（通常省略 `signature` 字段，或将其设为 `null`）。
- 其他字段（id、type、publisher、content、metadata、revision 等）均应包含在内。
- 具体哪些字段必须进入签名范围，以实现和规范的最终定义为准；v0.1 推荐将除 `signature` 外的所有已知字段纳入。

### 3.2 Canonicalization

必须使用 RFC 8785 JCS，确保不同实现产生完全相同的字节序列。详见 [03-canonicalization.md](03-canonicalization.md)。

### 3.3 签名值

Ed25519 签名结果为 64 字节。推荐以 **Base64URL（无填充）** 编码后放入 `signature.value`。

---

## 4. Signature 对象结构

```json
{
  "algorithm": "Ed25519",
  "keyId": "hxx:key:publisher:01",
  "value": "BASE64URL_SIGNATURE_VALUE"
}
```

| 字段 | 要求 | 说明 |
|------|------|------|
| `algorithm` | 必须 | 当前为 `Ed25519` |
| `keyId` | 必须 | 对应 Publisher 下的密钥标识 |
| `value` | 必须 | 签名值（Base64URL 推荐） |

---

## 5. 验证流程

```text
获取 Provenance Record
          ↓
提取 signature 对象，并暂时移除/清空 signature 字段
          ↓
对剩余 JSON 做 JCS Canonicalization
          ↓
根据 keyId 查找对应公钥
          ↓
使用公钥进行 Ed25519 验证
          ↓
同时检查：
  - 公钥是否属于声明的 Publisher
  - 密钥当前状态（是否已撤销）
  - Publisher 状态
  - 内容哈希是否匹配（如提供了原始内容）
```

验证结果应区分：

- 密码学签名是否有效
- 签署密钥 / Publisher 当前是否处于可信/可用状态

例如：

```text
Signature: VALID
Key Status: REVOKED
Publisher Status: ACTIVE
```

---

## 6. 与 Content Hash 的关系

- Content Hash 证明**内容本身**的完整性。
- Signature 证明**整条 Provenance Record**（包含 content hash、metadata、revision 等）由特定 Publisher 签署。

两者配合使用：

1. 验证签名 → 确认记录未被篡改且来自声称的 Publisher。
2. 验证 Content Hash → 确认实际拿到的内容与记录中声明的一致。

---

## 7. 实现要求

实现 MUST：

- 使用经过验证的 Ed25519 实现。
- 签名前严格执行 JCS。
- 正确处理 `keyId` 到公钥的解析。
- 支持历史密钥的验证（密钥轮换后旧签名仍可验证）。

实现 SHOULD：

- 对签名验证失败给出明确错误码与原因。
- 在验证结果中同时返回密码学结果与状态信息。

实现 MUST NOT：

- 将 Private Key 发送到 Registry 或任何远程服务（除非调用方明确使用自有的远程 KMS，且符合其安全模型）。
- 依赖不确定的 JSON 序列化行为。

---

## 8. 密钥泄露与应急

如果 Publisher 私钥泄露：

1. 立即将该 `keyId` 标记为 `revoked`。
2. 生成并注册新密钥。
3. 视情况对受影响的内容进行 `revoked` 或重新发布新 Revision。
4. 历史记录的签名仍然密码学有效，但验证器应提示密钥已撤销。

---

## 9. 测试向量

后续将在 `test-vectors/signature/` 提供官方测试向量，包括：

- 正常签名与验证
- 字段顺序不同但 JCS 后一致的情况
- 错误密钥、错误内容导致的失败案例
- 密钥轮换后的历史验证

---

## 10. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
