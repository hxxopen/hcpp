# HCPP Protocol — Status

**协议版本**：0.1  
**文档版本**：SPEC-0.1.0  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

Status 用于表达 Artifact 或特定 Revision 当前的生命周期状态。

HCPP 强调：

> 状态变更 ≠ 删除历史记录。

即使对象被撤销或撤回，其历史 Provenance Record 仍应保留并可验证。验证器应同时返回密码学验证结果与当前状态。

---

## 2. Artifact / Revision 状态取值

v0.1 定义以下状态：

| 状态 | 含义 |
|------|------|
| `active` | 当前有效、正常可用的版本 |
| `superseded` | 已被更新的版本替代 |
| `revoked` | 被发布者正式撤销 |
| `withdrawn` | 被撤回（通常由发布者主动发起） |

默认状态为 `active`。

---

## 3. 状态语义

### 3.1 active

- 表示该版本是当前对外主张的有效版本。
- 一个逻辑 Artifact 在同一时刻通常只有一个 `active` 版本（实现可强制此约束）。

### 3.2 superseded

- 表示存在更新的 Revision 取代了本版本。
- 旧版本的密码学签名与内容哈希仍然有效，但不再是“当前版本”。
- 验证器应提示用户存在更新版本，并尽量提供新版本 ID。

### 3.3 revoked

- 表示发布者明确撤销该内容/版本的效力主张。
- **不等于删除**。
- 常见原因：错误发布、合规要求、密钥泄露后的应急处理等。
- 验证结果示例：

```text
Signature: VALID
Content Hash: MATCHED
Status: REVOKED
```

### 3.4 withdrawn

- 与 `revoked` 接近，更强调“主动撤回”而非“因问题而撤销”。
- 实现可将两者合并处理，或保留细微语义差别供业务使用。

---

## 4. 状态转换（推荐状态机）

```text
          ┌─────────────┐
          │   active    │
          └──────┬──────┘
                 │
        ┌────────┼────────┐
        ▼                 ▼
┌───────────────┐   ┌─────────────┐
│  superseded   │   │   revoked   │
└───────────────┘   │  /withdrawn │
                    └─────────────┘
```

推荐规则：

- `active` → `superseded`：当新版本发布并声明替代旧版本时
- `active` → `revoked` / `withdrawn`：发布者主动撤销或撤回
- `superseded`、`revoked`、`withdrawn` 一般视为终态（v0.1 不强制支持恢复）

实现 MAY 支持有限的恢复路径（例如错误操作后的撤销恢复），但必须留下完整审计痕迹。

---

## 5. 状态与密码学验证的关系

状态是**业务/治理层**信息，密码学验证是**数学层**信息。

两者必须同时呈现，不可互相替代：

| 检查项 | 说明 |
|--------|------|
| Signature Valid | 签名是否密码学正确 |
| Content Matched | 内容哈希是否匹配 |
| Status | 当前生命周期状态 |

典型输出：

```json
{
  "valid": true,
  "contentMatched": true,
  "signatureValid": true,
  "status": "revoked",
  "revision": 1
}
```

这里的 `valid: true` 表示密码学层面通过；调用方需额外判断 `status` 是否满足自己的业务要求。

---

## 6. 在 Provenance Record 中的位置

Status 可作为 Provenance Record 的可选字段：

```json
{
  "hcpp": "0.1",
  "id": "HXX-ART-...",
  ...
  "status": "active"
}
```

也可以由 Registry 在查询时动态计算/附加（例如根据是否存在更新版本来判断 `superseded`）。

无论采用哪种方式，验证与查询接口都应能返回明确的状态信息。

---

## 7. Publisher 状态（相关但独立）

Publisher 自身也有状态（见 [04-publisher.md](04-publisher.md)）：

- `active`
- `suspended`
- `revoked`

Publisher 状态影响的是**新记录是否被接受**，以及验证时的信任提示，并不自动改变已存在内容的 status。

---

## 8. 实现要求

实现 MUST：

- 支持上述状态值（至少 `active`、`superseded`、`revoked`）
- 不因状态变更而删除历史记录
- 在验证结果中同时返回密码学结果与 status

实现 SHOULD：

- 在 supersede 时建立清晰的新旧版本关联
- 提供状态变更的审计日志
- 在用户界面明确区分“签名有效但已被撤销”的情况

实现 MUST NOT：

- 用删除记录的方式模拟 revoke / withdraw
- 仅因 status 非 active 就拒绝返回历史验证信息

---

## 9. 变更历史

- 2026-09-04：初始版本（v0.1 Draft）。
