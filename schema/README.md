# HCPP JSON Schemas

本目录包含 HCPP v0.1 的机器可读 JSON Schema 定义。

**状态**：Experimental（与协议 v0.1 Draft 同步）

## 文件说明

| 文件 | 说明 |
|------|------|
| `provenance-record.schema.json` | 完整的 Provenance Record（核心） |
| `publisher.schema.json` | Publisher 对象 |
| `artifact.schema.json` | Registry 返回的 Artifact 摘要 |
| `verification-result.schema.json` | 验证接口返回结果 |

## 使用建议

- 所有 Schema 基于 JSON Schema Draft 2020-12。
- 实现方可用这些 Schema 做输入校验与输出校验。
- Schema 会随协议次版本演进；破坏性变更只会在主版本发生。
- 正式的权威规范仍以 `protocol/` 目录下的 Markdown 文档为准。Schema 是辅助的机器可读形式。

## 在线 $id

当前使用的 `$id` 前缀为：

```text
https://hxxopen.github.io/hcpp/schema/
```

（实际 GitHub Pages 或文档站点就绪后可最终确认）

## 验证示例

使用任意支持 Draft 2020-12 的校验器（如 `ajv`、`jsonschema` 等）即可：

```bash
# 示例（Node.js + ajv-cli）
ajv validate -s provenance-record.schema.json -d my-record.json
```
