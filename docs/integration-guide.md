# HCPP 集成指南

**协议版本**：0.1  
**状态**：Draft  
**最后更新**：2026-09-04

---

## 1. 概述

本文面向希望将 HCPP 集成到业务系统（如 HxxNewsletter、内部 CMS、数据平台）的开发者，说明典型集成路径与注意点。

---

## 2. 集成模式

### 模式 A：SDK 直接集成（推荐）

业务服务嵌入 HCPP SDK：

1. 发布前计算 Content Hash、生成 ID、构造并签名 Provenance Record
2. 调用 Registry 完成注册
3. 将 HCPP ID 写入业务数据库并在页面展示

### 模式 B：通过 HXXBOT Skill

通过 HXXBOT 工具市场暴露的 HCPP 技能调用相同能力，适合已接入 HXXBOT 的应用或 Agent 场景。

### 模式 C：仅验证

只消费已有 HCPP 记录（查询 + 验证），不负责发布。

---

## 3. 发布侧集成步骤（以文章为例）

```text
1. 内容定稿（Markdown / HTML / PDF 等）
2. 按规范规范化并计算 Content Hash
3. 生成 HCPP ID
4. 填写 metadata（title、language、publishedAt 等）
5. 构造 Provenance Record
6. 使用 Publisher 私钥签名
7. 提交 Registry
8. 业务系统保存 HCPP ID，并完成正式发布
9. 在文章页展示 HCPP ID 与“验证”入口
```

关键点：

- Hash 计算必须与规范一致（去 BOM、换行归一化等）
- 私钥仅在安全环境中使用
- 先注册成功再对外公开，避免“页面有 ID 但 Registry 无记录”

---

## 4. 页面展示建议

在内容页页脚或侧边展示：

```text
HCPP Verified
HXX-ART-0198F7E3-ABCD-7EF0-89AB-CDEF01234567
Published: 2026-09-04
[Verify Provenance]
```

“Verify” 可跳转至验证网站或打开内嵌验证结果。

---

## 5. 验证侧集成

验证输入通常包括：

- HCPP ID
- 可选：当前拿到的内容字节（用于比对 Content Hash）

验证输出应包含：

- 签名是否有效
- Content Hash 是否匹配
- Publisher 信息
- Revision 与 Status
- （后续）Merkle / Anchor 结果

业务系统可根据 status 决定是否展示“已撤销”等警告。

---

## 6. 版本更新集成

当文章发生重要修改时：

1. 计算新 Content Hash
2. 创建新 Revision（previous 指向旧 ID 或旧 revision）
3. 签名并注册
4. 将旧版本标记为 superseded（如适用）
5. 更新业务侧“当前 HCPP ID”指向

---

## 7. 错误处理建议

| 场景 | 建议处理 |
|------|----------|
| Registry 不可用 | 发布流程失败并重试；不要静默跳过注册 |
| 签名失败 | 检查密钥与 canonicalization |
| Hash 不一致 | 检查规范化规则是否与规范一致 |
| 验证时 status=revoked | UI 明确提示，而不是当作普通成功 |

---

## 8. HxxNewsletter 参考路径

```text
CMS 点击发布
    → 调用 HCPP SDK / Skill
    → 完成 Hash、ID、签名、注册
    → 写回文章元数据中的 hcpp_id
    → 前端展示验证信息
```

同一套接口也可服务报告、数据集等其他类型。

---

## 9. 检查清单

发布前：

- [ ] Content Hash 规则正确
- [ ] ID 格式符合规范
- [ ] 签名使用正确 keyId
- [ ] Registry 返回成功
- [ ] 页面展示 HCPP ID

验证前：

- [ ] 能获取到 Record
- [ ] 能提供待比对内容（如需）
- [ ] 正确处理非 active 状态

---

## 10. 参考

- [protocol/02-content-hash.md](../protocol/02-content-hash.md)
- [protocol/05-signature.md](../protocol/05-signature.md)
- [protocol/06-provenance-record.md](../protocol/06-provenance-record.md)
- [architecture.md](architecture.md)
