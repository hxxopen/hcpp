# HCPP 部署指南

**协议版本**：0.1  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

本文档描述 HCPP Registry 及相关组件的常见部署方式，包括开发环境、生产环境与私有化部署的考虑。

v0.1 阶段以 **Reference Implementation** 为参考，实际生产部署可根据规模调整。

---

## 2. 组件清单

| 组件 | 说明 | v0.1 必要性 |
|------|------|-------------|
| Registry API 服务 | 处理注册、查询、验证 | 必须 |
| PostgreSQL | 主数据存储 | 必须 |
| SDK | 应用侧集成 | 必须 |
| 对象存储（可选） | 若需暂存原文或附件 | 可选 |
| 透明日志服务 | Merkle Tree 与 Proof | v0.2 |
| 锚定服务 | 将 Root 写入链或 TSA | v0.3 |
| 验证网站 | 面向用户的查询与验证 UI | 推荐 |

---

## 3. 开发 / 试验环境

快速启动建议：

- 单机 Docker Compose：Registry + PostgreSQL
- 本地生成密钥对用于测试 Publisher
- 使用 SDK 或 curl 完成创建与验证流程

重点验证：

- ID 生成格式
- Content Hash 规则（BOM、换行）
- 签名与验证闭环

---

## 4. 生产环境建议

### 4.1 计算与服务

- Registry 无状态部署，可水平扩展
- 前置 TLS 终止（负载均衡 / Ingress）
- 健康检查与就绪探针
- 结构化日志与指标（请求量、错误率、延迟）

### 4.2 数据库

- PostgreSQL 主从或托管服务
- 定期备份，并测试恢复
- 连接池与合理超时
- 对 artifacts / revisions 等大表考虑索引与分区策略

### 4.3 密钥与密钥管理

- Publisher 私钥**不要**与 Registry 部署在同一信任域（除非是完全自用的私有实例且可接受）
- 使用 KMS / HSM 管理签名密钥
- 轮换与撤销流程预先演练

### 4.4 网络与访问控制

- 全站 TLS
- 管理接口与公钥查询接口分离权限
- 速率限制与 API 配额
- 如有多租户，严格隔离 Publisher 权限

---

## 5. 私有化部署

组织可选择完全内网部署：

```text
内网应用 → 私有 HCPP Registry → 内网 PostgreSQL
```

特点：

- 数据不出边界
- 可自行管理 Publisher 与密钥策略
- 可选：定期将 Merkle Root 导出并锚定到公共基础设施（兼顾隐私与公共时间证明）

部署检查清单：

- [ ] PostgreSQL 高可用与备份
- [ ] TLS 与访问控制
- [ ] 密钥管理方案
- [ ] 监控与告警
- [ ] 升级与迁移流程
- [ ] 是否需要对接公共锚定

---

## 6. 混合模式

```text
私有 Registry（处理敏感内容）
        │
        │ 定期导出 Merkle Root
        ▼
公共锚定服务 / 区块链
```

适用于既需要内部保密，又希望获得外部时间戳证明的场景。

---

## 7. 配置与运维要点

- 明确协议版本与 Schema 版本
- 配置项外置（环境变量 / 配置中心）
- 数据库迁移使用版本化 migration
- 保留足够的审计日志（谁在何时注册/撤销了什么）
- 制定密钥泄露应急预案

---

## 8. 与官方托管服务的关系

协议本身开放。任何人都可以自建兼容 Registry。

未来可能提供的官方托管或企业支持（高可用、SLA、托管锚定等）属于商业选项，与协议开放性不冲突。

---

## 9. 参考

- [architecture.md](architecture.md)
- [security-considerations.md](security-considerations.md)
- [protocol/00-overview.md](../protocol/00-overview.md)
