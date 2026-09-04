# HCPP Protocol — Revision

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Revision 表示一个 Artifact 在某一时刻的不可变版本快照。

每次内容或关键元数据发生需要被追踪的变更时，都应产生一条新的 Revision（对应一条新的 Provenance Record），而不是修改旧记录。

HCPP 采用严格的 **append-only** 模型：

```text
Revision 1 → Revision 2 → Revision 3 → ...
```

旧 Revision 永远保留，可被查询与验证。

---

## 2. Revision 对象结构

在 Provenance Record 中的表示：

```json
{
  "revision": {
    "number": 1,
    "previous": null
  }
}
```

或后续版本：

```json
{
  "revision": {
    "number": 2,
    "previous": "HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567"
  }
}
```

### 字段说明

| 字段 | 类型 | 要求 | 说明 |
|------|------|------|------|
| `number` | integer | 必须 | 从 1 开始的正整数，同一 Artifact 下递增 |
| `previous` | string \| null | 必须 | 上一版本的 Object ID；首版本为 `null` |

---

## 3. 版本链规则

### 3.1 首个版本

- `number` 必须为 `1`
- `previous` 必须为 `null`

### 3.2 后续版本

- `number` 必须等于前一版本的 `number + 1`（推荐严格递增，不允许跳号）
- `previous` 必须指向前一个有效 Revision 的 `id`

### 3.3 链的完整性

实现 SHOULD 在注册新 Revision 时校验：

- `previous` 指向的记录确实存在
- 属于同一逻辑 Artifact（或明确允许跨 ID 的更新关系）
- `number` 连续且正确

---

## 4. 不可变原则

- 已发布的 Revision / Provenance Record **MUST NOT** 被修改。
- 任何更正、更新、撤回都通过新增 Revision 或更新状态完成。
- Registry 不得提供“覆盖写”接口覆盖历史记录。

---

## 5. 与 Artifact 的关系

- **Artifact**：逻辑对象（“这篇文档”），拥有稳定的主 ID 或逻辑标识。
- **Revision**：该对象在特定时间点的具体内容与 provenance 快照。

一个 Artifact 对应一条 Revision 链。当前有效版本通常是链上最新且状态为 `active` 的记录。

实现可以选择：

- 每个 Revision 使用全新的 HCPP ID（链式通过 `previous` 连接），或
- 保持同一逻辑 ID，仅通过 revision number 区分（需在数据模型中清晰表达）

v0.1 推荐在 Provenance Record 的 `id` 上体现每次发布的唯一性，并用 `previous` 显式链接。

---

## 6. 常见变更场景

| 场景 | 处理方式 |
|------|----------|
| 正文修改 | 新 Revision，更新 content hash |
| 仅标题/标签修改 | 新 Revision（或仅更新 metadata hash，视策略而定） |
| 发布者主动撤回 | 新状态 `withdrawn` 或 `revoked`，可伴随新 Revision |
| 被新版本替代 | 旧 Revision 标记为 `superseded`，新 Revision 成为 `active` |
| 纠正错误 | 新 Revision，可在 metadata 或扩展字段说明原因 |

---

## 7. 查询与历史

Registry 应支持：

- 获取某个 ID 对应的当前/最新记录
- 获取完整 Revision 历史（按 `number` 或时间排序）
- 根据 `previous` 向前/向后遍历版本链

API 示意（非规范性）：

```http
GET /v1/artifacts/{id}
GET /v1/artifacts/{id}/history
```

---

## 8. 验证时的 Revision 检查

完整验证除了检查签名与 content hash 外，还应：

- 确认 `revision.number` 与 `previous` 关系合理
- 确认当前状态（是否已被 supersede / revoke）
- 在结果中返回当前所处的版本号与状态

示例验证结果片段：

```json
{
  "valid": true,
  "contentMatched": true,
  "signatureValid": true,
  "revision": 2,
  "status": "active"
}
```

---

## 9. 实现要求

实现 MUST：

- 将每次需要追踪的变更作为新的不可变 Revision 处理
- 正确维护 `number` 与 `previous`
- 禁止修改已持久化的历史记录

实现 SHOULD：

- 提供便捷的历史查询接口
- 在注册时做基本的链完整性校验
- 清晰展示“当前有效版本”与历史版本的区别

---

## 10. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
