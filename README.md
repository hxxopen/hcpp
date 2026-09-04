# HCPP — HXX Content Provenance Protocol

**开放的数字内容唯一标识、来源证明、完整性验证、版本追踪与公共存证协议**

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-v0.1%20Draft-orange.svg)]()
[![Spec](https://img.shields.io/badge/Spec-Experimental-yellow.svg)]()

---

## 简介

HCPP（HXX Content Provenance Protocol）是一套面向数字内容的开放协议，用于为文章、报告、PDF、数据集、图片、视频及其他数字对象建立：

- 全球唯一标识（ID）
- 内容完整性指纹（Hash）
- 发布者身份与数字签名
- 发布时间与版本历史
- 内容引用关系
- 可验证的公共透明日志
- 可选的公共区块链锚定

**HCPP 不是公链，也不是 Token / NFT 协议。**

核心目标：

> 在不要求把内容本身写入区块链的前提下，让任何第三方都能够验证一个数字对象“是谁发布的、何时发布的、当时是什么内容、后来是否发生变化，以及该记录是否获得公共存证”。

HCPP 证明的是**可验证的来源关系、完整性和时间事实**，而不是内容事实真伪或法律版权归属。

---

## 当前状态

| 项目 | 状态 |
|------|------|
| 协议版本 | v0.1 Draft |
| 规范稳定性 | Experimental |
| 参考实现 | 规划中 |
| 生产就绪 | 否 |

第一阶段以 **HxxNewsletter** 为真实应用场景，并计划作为 **HXXBOT** 工具市场中的可调用技能对外提供能力。

---

## 核心设计原则

1. **ID 与 Hash 分离** — ID 标识对象，Hash 证明内容完整性
2. **Append-only** — 历史记录不可覆盖，只追加新 Revision
3. **默认不存储原文** — Registry 只保存指纹与 provenance
4. **区块链是可选 Anchor** — 第一阶段不依赖区块链
5. **密码学算法可替换** — 不将单一算法永久写死
6. **开放协议与实现分离** — 协议本身完全开放

---

## 快速概念

### Identifier（v0.1 锁定）

```text
HXX-{TYPE}-{UUIDv7}
```

- UUIDv7 **全部大写**
- **保留连字符**（8-4-4-4-12）

示例：

```text
HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567
HXX-RPT-0198F801-1234-7ABC-DEF0-9876543210FE
```

### Content Hash（v0.1 锁定）

- 算法：SHA-256
- 文本类：去除 UTF-8 BOM + 换行符统一为 LF，然后计算哈希
- 二进制类：直接对原始字节计算哈希
- **不做**空白折叠或任何语义归一化

### 推荐算法（v0.1）

| 用途           | 算法              |
|----------------|-------------------|
| Hash           | SHA-256           |
| Signature      | Ed25519           |
| Canonicalization | JCS (RFC 8785)  |
| Identifier     | UUIDv7            |

---

## 仓库结构

```text
hcpp/
├── LICENSE                 # Apache-2.0
├── README.md
├── VERSIONING.md           # 版本策略
├── CHANGELOG.md
├── protocol/               # 协议核心规范（权威来源）
├── schema/                 # JSON Schema
├── docs/                   # 详细文档
├── sdk/                    # 官方 SDK（Go / TypeScript / Python）
├── registry/               # Reference Implementation
├── verifier/
├── examples/
└── test-vectors/           # 官方测试向量
```

详细设计见 [设计文档](docs/design-v0.1.md)（或仓库内对应文件）。

---

## 版本路线图

| 版本   | 重点能力 |
|--------|----------|
| **v0.1** | ID、Hash、Publisher、Signature、Revision、基础 Registry、Verification |
| **v0.2** | Transparency Log、Merkle Tree、Merkle Proof |
| **v0.3** | Public Anchor（多链） |
| **v1.0** | DID / VC / C2PA 互操作、联邦 Registry、正式测试套件 |

完整版本策略见 [VERSIONING.md](VERSIONING.md)。

---

## 开发 SDK

计划提供多语言 SDK，优先顺序：

1. **Go**（与 Reference Registry 一致）
2. **TypeScript / JavaScript**
3. **Python**

SDK 将严格区分本地密码学能力与 Registry 网络调用，私钥始终由调用方管理。

---

## 私有化与商业化边界

- **协议规范、Schema、测试向量、基础 SDK**：Apache License 2.0，完全开放。
- 任何人都可以实现兼容的 Registry 与验证器。
- 官方托管服务、高可用私有化部署、企业级支持等，未来可能提供单独的商业选项。

---

## 贡献

我们欢迎对协议规范、测试向量、SDK 和参考实现的贡献。

请先阅读：

- [VERSIONING.md](VERSIONING.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)（待完善）
- [SECURITY.md](SECURITY.md)（待完善）

提交 Issue 或 Pull Request 前，请确保理解当前 v0.1 仍处于 Experimental 阶段，规范可能发生调整。

---

## License

本项目的协议规范、Schema、测试向量及官方 SDK 采用 [Apache License 2.0](LICENSE) 开源。

```text
Copyright 2026 HXX Open
```

---

## 相关链接

- 设计文档（v0.1 Draft）
- HXXBOT 工具市场：https://www.hxxbot.com/skills
- 问题反馈：GitHub Issues

---

**HCPP 不负责判断内容是真是假；HCPP 负责让内容的“来源、时间、完整性、版本和关系”变得可验证。**
