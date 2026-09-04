# HCPP 架构说明

**协议版本**：0.1  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 总体架构

HCPP 采用分层、可演进的设计，核心不依赖区块链。

```text
                         HCPP Ecosystem

     ┌─────────────────────────────────────────────┐
     │                Applications                 │
     │  HxxNewsletter · Reports · Data · Media     │
     │  HXXBOT Skills · 第三方应用                  │
     └──────────────────────┬──────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   HCPP SDK    │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Generate ID      Hash          Sign
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                    ┌───────────────┐
                    │ HCPP Registry │
                    └───────┬───────┘
                            │
                            ▼
                       PostgreSQL
                            │
                            ▼
                     Transparency Log
                            │
                            ▼
                       Merkle Tree
                            │
                            ▼
                       Merkle Root
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
            Public Anchor        Other Anchor
             (optional)           (optional)
```

---

## 2. 主要组件

### 2.1 Applications

业务系统（如 HxxNewsletter、内部 CMS、数据平台等）通过 SDK 或 HXXBOT Skill 调用 HCPP 能力。

### 2.2 HCPP SDK

提供本地与远程能力的统一接口：

- 本地：生成 ID、计算 Content Hash、构造与签名 Provenance Record
- 远程：向 Registry 注册、查询、验证

SDK 严格分离密码学操作与网络调用，私钥由调用方管理。

### 2.3 HCPP Registry

核心服务层，负责：

- ID 与 Publisher 管理
- Provenance Record 持久化
- Revision 历史
- Reference 关系
- 签名与状态查询
- 后续的 Transparency Log 与 Proof 服务

第一阶段推荐使用 PostgreSQL 作为主存储。

### 2.4 Transparency Log（v0.2+）

Append-only 日志 + Merkle Tree，用于生成可公开验证的包含证明（Merkle Proof）。

### 2.5 Public Anchor（v0.3+）

将周期性 Merkle Root 锚定到公共区块链或其他时间戳服务，提供独立于 Registry 运营方的外部时间证明。

---

## 3. 数据流（典型发布）

```text
内容准备完成
    │
    ▼
规范化（去 BOM、换行归一化等）
    │
    ▼
计算 Content Hash（SHA-256）
    │
    ▼
生成 HCPP ID（UUIDv7）
    │
    ▼
构造 Provenance Record
    │
    ▼
JCS Canonicalization + Ed25519 签名
    │
    ▼
提交至 Registry
    │
    ▼
业务系统完成正式发布
```

验证时路径相反：拉取 Record → 验证签名 → 比对 Content Hash → 检查 Revision 与 Status →（可选）验证 Merkle Proof 与 Anchor。

---

## 4. 信任边界

| 层级 | 信任来源 | 说明 |
|------|----------|------|
| 密码学层 | 哈希与签名 | 可独立验证，不依赖 Registry 诚实 |
| Registry 层 | 运营方 | 提供存储、查询、历史与状态 |
| 透明日志层 | Merkle 结构 | 可证明某记录被包含在某 Root 中 |
| 锚定层 | 公共区块链 / TSA | 提供外部时间与不可篡改证明 |

即使 Registry 被攻破，已有的签名与哈希仍可被第三方验证；透明日志与锚定进一步降低对单一运营方的信任依赖。

---

## 5. 部署形态

### 5.1 公共 Registry

由 HXX 或社区运营，面向多租户 / 多 Publisher。

### 5.2 私有化部署

组织内部独立部署 Registry，数据不出内网。可选择是否连接公共透明日志或锚定服务。

### 5.3 混合模式

私有 Registry 处理敏感内容，定期将 Merkle Root 锚定到公共基础设施，兼顾隐私与公共可验证性。

---

## 6. 与 HXXBOT 的关系

HCPP 能力将作为 HXXBOT 工具市场中的 Skill 对外暴露，使 HxxNewsletter 及其他应用可通过统一技能接口调用生成、哈希、签名、验证等功能。

---

## 7. 演进原则

- 先做本地可验证的 Provenance（v0.1）
- 再做透明日志（v0.2）
- 最后做公共锚定（v0.3）
- 协议层保持稳定，实现与部署方式可灵活演进

---

## 8. 参考

- [protocol/00-overview.md](../protocol/00-overview.md)
- [VERSIONING.md](../VERSIONING.md)
