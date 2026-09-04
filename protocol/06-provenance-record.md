# HCPP Protocol — Provenance Record

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Provenance Record 是 HCPP 的核心数据对象。它把一个数字对象在某一时刻的身份、内容指纹、发布者、版本信息与签名等绑定在一起，形成可验证的来源记录。

每条 Provenance Record 对应一次不可变的发布或修订事件。

---

## 2. 最小结构（v0.1）

```json
{
  "hcpp": "0.1",
  "id": "HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567",
  "type": "article",

  "publisher": {
    "id": "hxx:publisher:hxxnewsletter"
  },

  "content": {
    "mediaType": "text/markdown",
    "encoding": "utf-8",
    "canonicalization": "bom-strip+lf-normalize",
    "hash": {
      "algorithm": "SHA-256",
      "value": "4f9c8a2b1e3d5f67890123456789abcdef0123456789abcdef0123456789abcd"
    }
  },

  "metadata": {
    "title": "China Semiconductor Supply Chain Outlook 2026",
    "language": "en",
    "createdAt": "2026-09-04T08:20:00Z",
    "publishedAt": "2026-09-04T08:30:00Z"
  },

  "revision": {
    "number": 1,
    "previous": null
  },

  "signature": {
    "algorithm": "Ed25519",
    "keyId": "hxx:key:publisher:01",
    "value": "..."
  }
}
```

---

## 3. 字段详解

### 3.1 必填字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `hcpp` | string | 协议版本，v0.1 固定为 `"0.1"` |
| `id` | string | Object Identifier，见 [01-identifier.md](01-identifier.md) |
| `type` | string | 对象类型（article、report、pdf 等） |
| `publisher` | object | 发布者引用，至少包含 `id` |
| `content` | object | 内容描述与哈希，见 [02-content-hash.md](02-content-hash.md) |
| `revision` | object | 版本信息 |
| `signature` | object | 数字签名，见 [05-signature.md](05-signature.md) |

### 3.2 推荐字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `metadata` | object | 标题、语言、时间等描述性信息 |
| `status` | string | `active` / `superseded` / `revoked` / `withdrawn` |

### 3.3 content 对象

```json
{
  "mediaType": "text/markdown",
  "encoding": "utf-8",
  "canonicalization": "bom-strip+lf-normalize",
  "hash": {
    "algorithm": "SHA-256",
    "value": "..."
  }
}
```

### 3.4 revision 对象

```json
{
  "number": 1,
  "previous": null
}
```

或：

```json
{
  "number": 2,
  "previous": "HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567"
}
```

- `number`：从 1 开始的正整数
- `previous`：上一版本的 ID；首版本为 `null`

### 3.5 signature 对象

见 [05-signature.md](05-signature.md)。

---

## 4. 签名范围

对 Provenance Record 签名时：

1. 构造完整记录，但**省略或清空 `signature` 字段**。
2. 使用 JCS（RFC 8785）进行 canonicalization。
3. 对结果进行 Ed25519 签名。
4. 将签名结果写回 `signature` 字段。

详细流程见签名规范。

---

## 5. 不可变性

- 一条已发布的 Provenance Record **MUST NOT** 被修改。
- 任何内容或元数据的变更都应产生新的 Revision（新的 Record）。
- Registry 应以 append-only 方式保存历史。

---

## 6. 与 Artifact 的关系

- **Artifact** 是逻辑对象（“这篇文档”）。
- **Provenance Record / Revision** 是该对象在某一时刻的不可变快照与证明。

一个 Artifact 可以对应多条按时间排序的 Provenance Record（Revision 链）。

---

## 7. Metadata 与 Content 的分离建议

推荐同时维护：

- `content.hash`（或独立的 `contentHash`）— 仅覆盖正文/二进制
- `metadata` 的哈希（`metadataHash`）— 覆盖标题、标签、时间等

这样仅修改标题时不会被误判为正文被篡改。验证结果应分别返回 `contentMatched` 与 `metadataMatched`。

---

## 8. 状态字段

`status` 可选，取值：

| 值 | 含义 |
|----|------|
| `active` | 当前有效版本 |
| `superseded` | 已被更新版本替代 |
| `revoked` | 被发布者撤销 |
| `withdrawn` | 撤回 |

注意：`revoked` / `withdrawn` **不等于删除记录**。历史记录应继续可查询与验证，验证器同时报告密码学结果与当前状态。

---

## 9. 扩展性

- 允许增加可选字段（次版本兼容）。
- 新增字段不应破坏已有签名的可验证性（旧记录按旧结构验证）。
- 通过 `"hcpp": "0.1"` 区分协议版本。

未来可能增加的字段示例：

- `generation`（AI 生成声明）
- `references`（引用列表）
- `anchor`（公共锚定信息）
- DID / VC 相关绑定

---

## 10. JSON Schema

机器可读定义见：

```text
schema/provenance-record.schema.json
```

Schema 是辅助工具，**规范性要求以本文件及关联 protocol 文档为准**。

---

## 11. 实现要求摘要

实现 MUST：

- 生成符合本结构的 Record
- 正确计算 content hash 并填写
- 使用 JCS + Ed25519 进行签名
- 将 `hcpp` 字段设为 `"0.1"`
- 保证已发布 Record 不可变

实现 SHOULD：

- 支持 metadata 与 content 哈希分离
- 在验证结果中清晰区分各类检查项

---

## 12. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
