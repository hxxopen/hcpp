# Contributing to HCPP

感谢你对 HCPP（HXX Content Provenance Protocol）的兴趣。

本项目目前处于 **v0.1 Draft / Experimental** 阶段。我们欢迎对协议规范、Schema、文档、测试向量和参考实现的贡献，但请先阅读以下说明。

---

## 行为准则

参与本项目即表示你同意遵守 [Code of Conduct](CODE_OF_CONDUCT.md)（待完善时以社区通用准则为准）。请保持友善、建设性的沟通。

---

## 贡献前请先了解

1. **协议规范是权威来源**  
   规范性要求以 `protocol/` 目录下的文档为准。`docs/` 是解释性文档，`schema/` 是机器可读辅助定义。

2. **版本策略**  
   请阅读 [VERSIONING.md](VERSIONING.md)。v0.1 仍可能发生调整；破坏性变更会记录在 [CHANGELOG.md](CHANGELOG.md)。

3. **当前优先事项**  
   - 澄清与完善 v0.1 规范文本
   - 补充官方测试向量（test-vectors）
   - 改进文档可读性与示例
   - 后续：参考实现与多语言 SDK

---

## 如何贡献

### 报告问题

- 使用 GitHub Issues。
- 报告规范歧义、错误、安全相关疑虑（安全问题请优先走 [SECURITY.md](SECURITY.md) 流程）或改进建议。
- 请尽量说明：相关文件/章节、问题描述、期望行为、可能的影响。

### 改进文档或规范

1. Fork 仓库并创建分支。
2. 修改对应的 `protocol/` 或 `docs/` 文件。
3. 若涉及行为变更，同步更新：
   - `CHANGELOG.md`
   - 相关 JSON Schema（如有）
   - 示例或测试向量（如有）
4. 提交 Pull Request，并在描述中说明动机与影响范围。

### 贡献 Schema

- Schema 应与 `protocol/` 保持一致。
- 使用 JSON Schema Draft 2020-12。
- 变更时请说明兼容性影响。

### 贡献测试向量

测试向量对互操作性至关重要。欢迎提供：

- Content Hash 边界情况（BOM、换行符、空内容等）
- JCS 规范化案例
- 签名与验证案例

请将向量放在清晰的目录结构中，并附简短说明。

### 贡献代码（SDK / Registry）

- 目前参考实现与 SDK 仍在早期阶段。
- 提交代码前请确保：
  - 通过基本测试
  - 风格与现有代码大致一致
  - 不引入不必要的依赖
  - 私钥处理符合安全要求（不得将私钥上传 Registry）

---

## Pull Request 建议

- 标题简明，说明“做了什么”。
- 描述中说明“为什么”以及影响范围（规范 / 文档 / 实现）。
- 关联相关 Issue（如有）。
- 保持 PR 聚焦；大改动可拆分。
- 若修改规范，请明确是否为破坏性变更。

---

## 开发与文档约定

- 协议文档使用 Markdown。
- 规范性语句可使用 MUST / SHOULD / MAY（参考 RFC 2119 语义）。
- 中英文可根据具体文件既有风格保持一致；新增内容优先与同目录风格统一。
- 提交信息建议清晰、使用现在时，例如：`Add revision status examples`。

---

## 设计讨论

重大设计变更建议先开 Issue 讨论，达成基本共识后再提交大范围 PR，以减少返工。

---

## 许可

贡献内容默认与本项目相同，采用 [Apache License 2.0](LICENSE)。  
你提交贡献即表示同意以该许可授权给项目使用。

---

## 问题与联系

- 一般问题：GitHub Issues
- 安全问题：[SECURITY.md](SECURITY.md)

再次感谢你的贡献。
