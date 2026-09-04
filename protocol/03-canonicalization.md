# HCPP Protocol — Canonicalization

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

密码学操作（哈希与签名）要求输入数据具有**确定性的字节表示**。

同一逻辑内容如果因为字段顺序、空白、换行或编码差异而产生不同字节，就会导致哈希和签名不一致。因此 HCPP 对需要参与哈希或签名的数据定义明确的 Canonicalization 规则。

---

## 2. 适用范围

| 数据类型 | 规范化方法 | 参考文档 |
|----------|------------|----------|
| 普通文本（Markdown、HTML、Plain Text 等） | 去 BOM + 换行归一化为 LF | [02-content-hash.md](02-content-hash.md) |
| 二进制文件（PDF、图片、音视频等） | 不做规范化，直接使用原始字节 | [02-content-hash.md](02-content-hash.md) |
| JSON（参与签名或作为规范内容时） | RFC 8785 JCS | 本文档 |
| Provenance Record 本身（签名前） | RFC 8785 JCS | 本文档 |

---

## 3. JSON Canonicalization（RFC 8785）

HCPP v0.1 对需要确定性表示的 JSON 数据采用 **JSON Canonicalization Scheme (JCS)**。

规范：https://www.rfc-editor.org/rfc/rfc8785.html

### 3.1 必须遵守的规则

对参与 hash 或 signature 的 JSON：

- MUST 使用 UTF-8 编码
- MUST 消除重复的对象成员名（按 JCS 规则处理）
- MUST 对对象成员进行确定性排序（字典序）
- MUST 按照 JCS 规定的方式序列化数字、字符串、数组、对象等
- MUST NOT 在 canonical 表示上再做额外的空白或格式化修改

### 3.2 常见要求摘要

- 对象键按字典序排序
- 无多余空白
- 数字使用最短的满足精度的表示（遵循 JCS）
- 字符串正确处理转义

实现应使用经过验证的 JCS 库，而不是自行实现完整规则。

---

## 4. 文本内容的 Canonicalization

详见 [02-content-hash.md](02-content-hash.md) 第 3.1 节。

简要回顾：

1. 去除 UTF-8 BOM（`EF BB BF`）
2. 将 `\r\n` 和 `\r` 统一替换为 `\n`
3. 不做空白折叠、trim 或语义级处理
4. 按 UTF-8 字节参与后续哈希

对应的 `canonicalization` 声明值：

```text
bom-strip+lf-normalize
```

---

## 5. 二进制内容

二进制内容（PDF、图片、音视频等）：

- `canonicalization` 声明为 `none`（或不填）
- 直接对原始字节计算哈希
- MUST NOT 进行任何修改

---

## 6. Provenance Record 的签名输入

对 Provenance Record 进行数字签名时：

1. 构造完整的待签名 JSON 对象（通常**不包含** `signature` 字段本身，或将 signature 字段置为空/省略，具体以签名规范为准）。
2. 使用 JCS 对其进行 canonicalization。
3. 对 canonical 后的 UTF-8 字节进行签名（或先哈希再签名，取决于签名方案细节）。

完整签名流程见 [05-signature.md](05-signature.md)。

---

## 7. Content Descriptor 中的声明

为了让验证方知道实际使用了哪种规范化方法，推荐在 Content Descriptor 中显式声明：

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

允许的取值（v0.1）：

| 值 | 含义 |
|----|------|
| `bom-strip+lf-normalize` | 文本默认处理 |
| `none` | 无额外处理（主要用于二进制） |
| `jcs` 或 `rfc8785` | JSON Canonicalization Scheme |

---

## 8. 实现要求

实现 MUST：

- 对 JSON 使用符合 RFC 8785 的 JCS 实现
- 对文本严格遵守去 BOM + LF 归一化顺序
- 在 Descriptor 中准确声明所使用的 canonicalization 方法

实现 SHOULD：

- 使用成熟、有测试覆盖的 JCS 库
- 提供清晰的错误信息（当输入无法被正确规范化时）

实现 MUST NOT：

- 在计算哈希或签名前进行“智能”格式化或语义清理
- 依赖特定编程语言的默认 JSON 序列化行为（必须显式使用 JCS）

---

## 9. 测试向量

后续将在 `test-vectors/canonicalization/` 提供官方测试向量，覆盖：

- 不同键序的 JSON 对象
- 含 BOM / 不含 BOM 的文本
- 混合换行符的文本
- 空对象、空数组、特殊数字与字符串

所有实现应以通过官方测试向量作为基本兼容性要求。

---

## 10. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
