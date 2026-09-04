# Content Hash Test Vectors

Vectors for [protocol/02-content-hash.md](../../protocol/02-content-hash.md).

## Rules under test (v0.1)

Text content:

1. Strip UTF-8 BOM (`EF BB BF`) if present
2. Normalize newlines: `\r\n` → `\n`, `\r` → `\n`
3. No whitespace folding, no trim, no Unicode normalization
4. Hash UTF-8 bytes with SHA-256

Binary content:

- Hash raw bytes with no modification

## Case status

| ID | Description | Status |
|----|-------------|--------|
| CH-001 | Plain UTF-8 text, LF only, no BOM | Done |
| CH-002 | Same text with UTF-8 BOM | Done |
| CH-003 | CRLF newlines | Done |
| CH-004 | Mixed CR / CRLF / LF | Done |
| CH-005 | Empty content | Done |
| CH-006 | Leading/trailing whitespace preserved | Done |
| CH-007 | Binary file (minimal 1x1 PNG, input.bin) | Done |
| CH-008 | JSON as text (not JCS — raw text rules) | Planned |

**Note:** CH-001 / CH-002 / CH-003 share the same expected hash after normalization (same logical content).

## Case format

```text
content-hash/
  CH-00x/
    input.txt          # or input.bin
    meta.json          # mediaType, canonicalization, notes
    expected.json      # { "algorithm": "SHA-256", "value": "..." }
```

`meta.json` example:

```json
{
  "id": "CH-001",
  "mediaType": "text/plain",
  "canonicalization": "bom-strip+lf-normalize",
  "description": "Plain UTF-8 text with LF newlines, no BOM"
}
```
