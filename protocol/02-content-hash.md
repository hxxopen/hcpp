# HCPP Protocol — Content Hash

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Content Hash 用于证明数字对象在某一时刻的内容完整性。

它回答的问题是：

> “这个对象的内容有没有发生变化？”

Content Hash 与 Object Identifier 严格分离：

```text
Object ID   ≠   Content Hash
```

---

## 2. 算法

v0.1 推荐（也是当前默认）算法：

```text
SHA-256
```

输出表示推荐使用小写十六进制，并加上算法前缀（可选但推荐）：

```text
sha256:4f9c8a2b1e3d5f67890123456789abcdef0123456789abcdef0123456789abcd
```

或仅使用原始十六进制字符串（实现间需约定一致）。

未来可扩展支持 SHA-512、SHA-3、BLAKE3 等，通过在 Descriptor 中明确声明算法实现可替换。

---

## 3. 规范化规则（v0.1 规范性要求）

**核心原则：纯字节级确定性处理，不做任何语义归一化。**

### 3.1 文本类内容

适用于 `text/*`、`application/json`、`application/xml`、`application/javascript` 等以字符为主的类型。

处理步骤（必须按顺序执行）：

1. **去除 UTF-8 BOM**  
   如果内容以字节序列 `EF BB BF` 开头，则移除这三字节。

2. **换行符归一化**  
   - 将所有 `\r\n`（CRLF）替换为 `\n`（LF）  
   - 将所有单独的 `\r`（CR）替换为 `\n`（LF）  
   - 最终只保留 `\n` 作为换行符

3. **不做其他处理**  
   - MUST NOT 进行空白折叠  
   - MUST NOT trim 首尾空白  
   - MUST NOT 改变空格、制表符数量  
   - MUST NOT 进行 Unicode 正规化（NFC/NFD 等）  
   - MUST NOT 进行 HTML/Markdown 语义级清理

4. 将处理后的结果按 **UTF-8** 编码为字节序列，计算 SHA-256。

### 3.2 二进制内容

适用于 `application/pdf`、`image/*`、`video/*`、`audio/*` 以及任何非文本类型。

- 直接对**原始文件字节**计算 SHA-256。
- MUST NOT 做任何修改（包括不去除可能存在的 BOM 或其他头）。

### 3.3 结构化 JSON（特殊规则）

当内容本身是 JSON，并且需要参与签名或作为规范内容时：

- MUST 先按照 **RFC 8785 JSON Canonicalization Scheme (JCS)** 进行 canonicalization。
- 然后再对 canonical 后的 UTF-8 字节计算 SHA-256。

普通文本 Markdown / HTML **不**走 JCS，只走 3.1 的规则。

---

## 4. Content Descriptor

推荐在 Provenance Record 中使用以下结构描述内容：

```json
{
  "mediaType": "text/markdown",
  "encoding": "utf-8",
  "canonicalization": "bom-strip+lf-normalize",
  "hash": {
    "algorithm": "SHA-256",
    "value": "4f9c8a2b1e3d5f67890123456789abcdef0123456789abcdef0123456789abcd"
  }
}
```

字段说明：

| 字段 | 要求 | 说明 |
|------|------|------|
| `mediaType` | 推荐 | IANA media type |
| `encoding` | 文本类推荐 | 目前主要为 `utf-8` |
| `canonicalization` | 推荐 | 记录实际使用的规范化方法 |
| `hash.algorithm` | 必须 | 当前为 `SHA-256` |
| `hash.value` | 必须 | 哈希值（十六进制） |

常见 `canonicalization` 取值：

- `bom-strip+lf-normalize` — 文本类默认
- `none` — 二进制或已确定无需处理
- `jcs` / `rfc8785` — JSON Canonicalization

---

## 5. 与 Metadata Hash 的分离

强烈建议将 **内容哈希** 与 **元数据哈希** 分开计算与存储：

```json
{
  "contentHash": "sha256:...",
  "metadataHash": "sha256:..."
}
```

- `contentHash`：仅覆盖正文 / 二进制载荷。
- `metadataHash`：覆盖 title、language、tags、publishedAt（声明值）、generation 信息等。

这样当仅修改标题或标签时，不会被误判为正文被篡改。

验证结果应分别返回：

```json
{
  "contentMatched": true,
  "metadataMatched": false
}
```

---

## 6. 实现要求

实现 MUST：

- 严格遵守第 3 节的规范化顺序与规则。
- 在 Descriptor 中准确声明所使用的算法与 canonicalization 方法。
- 对同一输入字节序列始终产生相同的哈希值。

实现 SHOULD：

- 提供清晰的错误信息（例如不支持的 mediaType 或编码）。
- 在测试中覆盖 BOM 存在/不存在、CRLF/CR/LF 混合等边界情况。

实现 MUST NOT：

- 根据“看起来更干净”而额外修改内容。
- 在计算哈希前进行 HTML 净化、Markdown 渲染或智能空白处理。

---

## 7. 安全性考虑

- SHA-256 目前对于内容完整性用途是可接受的。
- 协议设计允许未来算法升级；已存在的记录继续使用其声明的算法进行验证。
- 哈希碰撞风险由所选算法的强度保证，实现不应自行截断哈希值。

---

## 8. 测试向量（待补充）

后续将在 `test-vectors/content-hash/` 目录提供官方测试向量，覆盖：

- 无 BOM / 有 BOM 的文本
- 不同换行符风格
- 纯二进制文件
- JSON + JCS 案例
- 边界长度与空内容

所有实现都应以通过官方测试向量作为兼容性的基本要求。

---

## 9. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。锁定“去 BOM + LF 归一化、纯字节处理、不做语义归一化”的规则。
