# HCPP Protocol — Reference

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Reference 用于表达 HCPP 对象之间的关系，形成可验证的内容关系图（Reference Graph）。

典型用途：

- 文章引用数据源或报告
- 报告派生自某个数据集
- 新版本更新/替代旧版本
- 翻译、摘要、回应等衍生关系

---

## 2. 基本结构

```json
{
  "source": "HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567",
  "target": "HXX-SRC-0198F801-1234-7ABC-DEF0-9876543210FE",
  "relation": "cites",
  "createdAt": "2026-09-04T09:00:00Z"
}
```

### 字段说明

| 字段 | 类型 | 要求 | 说明 |
|------|------|------|------|
| `source` | string | 必须 | 关系发出方的 HCPP ID |
| `target` | string | 必须 | 关系指向方的 HCPP ID |
| `relation` | string | 必须 | 关系类型 |
| `createdAt` | string | 推荐 | 关系创建时间（RFC 3339） |

---

## 3. 推荐关系类型（v0.1）

| relation | 含义 |
|----------|------|
| `cites` | 引用 |
| `references` | 参考 |
| `derived-from` | 派生自 |
| `updates` | 更新 |
| `supersedes` | 替代（版本层面） |
| `translates` | 翻译自 |
| `summarizes` | 摘要自 |
| `responds-to` | 回应 |

- 关系类型应保持稳定、语义清晰。
- 新增关系类型属于次版本可兼容变更。
- 实现 MAY 支持自定义关系类型，但官方互操作以推荐列表为准。

---

## 4. 方向性

所有关系均为**有向**：

```text
source  --relation-->  target
```

示例：

```text
Article  --cites-->  Source
Report   --derived-from-->  Dataset
Rev2     --supersedes-->  Rev1
```

目前不强制定义逆关系；需要双向查询时由实现自行维护或计算。

---

## 5. 与 Revision / Status 的关系

- `supersedes` 关系常与 Status 中的 `superseded` 配合使用。
- 一个对象被新版本替代时，除了更新旧记录的 status，还可显式建立 `supersedes` 引用，便于图谱遍历。

---

## 6. 存储与查询

Reference 可以作为：

- 独立的关系记录存储在 Registry 中，或
- 嵌入在 Provenance Record 的扩展字段中

v0.1 更推荐前者（独立存储），以便灵活查询与形成图谱。

Registry 应支持的基本查询（示意）：

```http
GET /v1/artifacts/{id}/references          # 出边
GET /v1/artifacts/{id}/referenced-by      # 入边
```

---

## 7. 可验证性

v0.1 中，Reference 本身可以是：

- 由 Publisher 声明并随 Provenance Record 一起签名，或
- 由 Registry 记录的辅助元数据

为提高可信度，推荐将关键引用关系纳入被签名的 Provenance Record（或单独签名的关系断言）中。具体机制可在后续版本进一步规范化。

---

## 8. 使用示例（HxxNewsletter）

```text
HXX-ART-001
 ├── cites          → HXX-SRC-001
 ├── cites          → HXX-SRC-002
 └── derived-from  → HXX-DATA-001
```

未来可扩展为更复杂的供应链 / 知识图谱。

---

## 9. 实现要求

实现 SHOULD：

- 支持推荐的关系类型
- 提供按 source / target 查询引用的能力
- 保持关系记录的不可变性（或至少可审计）

实现 MAY：

- 支持自定义 relation
- 构建可视化的 Reference Graph
- 对循环引用进行检测或限制

---

## 10. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
