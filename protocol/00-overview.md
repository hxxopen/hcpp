# HCPP Protocol — Overview

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 引言

HCPP（HXX Content Provenance Protocol）是一套开放的数字内容 provenance 协议。

它为数字对象提供：

- 全球唯一标识（Object Identifier）
- 内容完整性指纹（Content Hash）
- 发布者身份与数字签名
- 版本历史（Revision）
- 对象间引用关系（Reference）
- 可选的透明日志与公共锚定

HCPP **不是**区块链，也不是 Token / NFT 协议。区块链仅作为可选的外部时间锚定（Anchor）。

---

## 2. 设计目标

### 2.1 核心问题

任何第三方应能独立验证：

1. **What** — 这是什么对象？
2. **Who** — 谁发布的？
3. **When** — 什么时候发布的？
4. **Integrity** — 内容是否被修改过？
5. **History** — 经历过哪些版本？
6. **Proof** — 是否存在独立于发布者的公共证明？

### 2.2 非目标

HCPP 明确不证明：

- 内容事实的真伪
- 发布者没有错误或恶意
- 引用来源本身的可信度
- 法律意义上的版权归属
- 某个发布者天然值得信任

HCPP 只提供**可验证的来源、完整性、时间与版本关系**。

---

## 3. 核心设计原则

1. **ID 与 Hash 分离**  
   ID 标识对象身份，Hash 证明内容完整性。

2. **Append-only**  
   历史 Revision 不可覆盖、不可修改。

3. **默认不存储原文**  
   Registry 保存指纹与 provenance，不强制保存正文。

4. **区块链是可选 Anchor**  
   第一阶段不依赖区块链；链上只锚定 Merkle Root。

5. **密码学算法可替换**  
   不将单一算法永久写死，通过字段声明实际使用的算法。

6. **开放协议与实现分离**  
   协议本身（规范、Schema、测试向量）采用 Apache-2.0；具体托管服务与私有化支持可另行提供。

---

## 4. 对象模型

```text
HCPP Object
├── Artifact          # 被标识的数字内容
├── Publisher         # 发布主体
├── Revision          # 不可变的版本记录
├── Reference         # 对象间关系
└── Proof             # 签名、Merkle Proof、Anchor Proof 等
```

详细定义见后续章节。

---

## 5. 协议版本

每条 Provenance Record 必须包含：

```json
{
  "hcpp": "0.1"
}
```

验证器根据此字段选择对应的解析与校验逻辑，并应支持多版本并行。

版本策略详见仓库根目录 [VERSIONING.md](../VERSIONING.md)。

---

## 6. 规范文档结构

| 文档 | 内容 |
|------|------|
| [00-overview.md](00-overview.md) | 本概述 |
| [01-identifier.md](01-identifier.md) | Object Identifier |
| [02-content-hash.md](02-content-hash.md) | Content Hash 与规范化 |
| [03-canonicalization.md](03-canonicalization.md) | JSON Canonicalization 等 |
| [04-publisher.md](04-publisher.md) | Publisher 身份与密钥 |
| [05-signature.md](05-signature.md) | 数字签名 |
| [06-provenance-record.md](06-provenance-record.md) | Provenance Record 完整结构 |
| [07-revision.md](07-revision.md) | Revision 模型 |
| [08-reference.md](08-reference.md) | Reference 关系 |
| [09-status.md](09-status.md) | 状态机 |
| [10-transparency-log.md](10-transparency-log.md) | 透明日志（v0.2+） |
| [11-anchoring.md](11-anchoring.md) | 公共锚定（v0.3+） |
| [99-versioning.md](99-versioning.md) | 协议版本细节 |

机器可读 Schema 见 [`../schema/`](../schema/)。

---

## 7. 推荐算法（v0.1）

| 用途              | 算法 / 规范          |
|-------------------|----------------------|
| Hash              | SHA-256              |
| Signature         | Ed25519              |
| JSON Canonicalization | RFC 8785 (JCS)   |
| Identifier        | UUIDv7（大写 + 连字符） |

---

## 8. 与现有标准的关系

HCPP 刻意与以下标准对齐或预留兼容空间：

- RFC 8785 — JSON Canonicalization Scheme
- W3C Verifiable Credentials
- Decentralized Identifiers (DID)
- C2PA（内容来源与媒体 provenance）
- Merkle Tree / 透明日志实践
- 公共区块链时间锚定

HCPP 的目标是成为连接这些标准的**内容 provenance 与信任层**，而不是重复实现它们。

---

## 9. 实现状态与路线图

| 版本  | 重点 |
|-------|------|
| v0.1  | ID、Hash、Publisher、Signature、Revision、基础 Registry、Verification |
| v0.2  | Transparency Log、Merkle Proof |
| v0.3  | Public Anchor（多链） |
| v1.0  | DID / VC / C2PA、联邦 Registry、正式测试套件 |

当前（v0.1）处于 **Draft / Experimental** 阶段，规范仍可能调整。

---

## 10. 安全与信任边界

- 密码学证明的是“某私钥对某条记录的签名”以及“内容哈希匹配”。
- 组织信任（某个 Publisher 是否值得信任）需要额外的身份验证与信任体系，不属于 v0.1 范围。
- Private Key 永远不应上传至 Registry。

更完整的威胁模型见 `docs/threat-model.md`（待完善）。

---

## 11. 许可

本协议规范、配套 Schema 与测试向量采用 **Apache License 2.0** 开源。

---

**核心原则回顾**：

> HCPP 不负责判断内容是真是假；HCPP 负责让内容的“来源、时间、完整性、版本和关系”变得可验证。
