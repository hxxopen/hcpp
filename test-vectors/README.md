# HCPP Test Vectors

Official test vectors for HCPP protocol interoperability.

**Status**: Skeleton for v0.1 — concrete vectors will be added incrementally.

## Purpose

Test vectors allow independent implementations to verify they produce identical:

- Object Identifiers (format)
- Content Hashes (after canonicalization)
- JCS canonicalization results
- Signatures and verification outcomes

Passing the official vectors is the primary compatibility check for HCPP implementations.

## Directory layout

```text
test-vectors/
├── README.md                 # this file
├── content-hash/
│   ├── README.md
│   └── (cases: input files + expected hash)
├── canonicalization/
│   ├── README.md
│   └── (JSON inputs + expected JCS output)
├── identifier/
│   ├── README.md
│   └── (generation / parsing examples)
└── signature/
    ├── README.md
    └── (keys, records, expected signatures — test keys only)
```

## Conventions

- Each case should be self-contained and documented.
- Prefer deterministic, small inputs.
- Never include real production private keys.
- Use only test key pairs generated for vectors.
- Hash values: lowercase hex unless a case explicitly tests encoding variants.
- When a case depends on protocol version, state it clearly (`hcpp: "0.1"`).

## Adding new vectors

1. Place inputs and expected outputs in the appropriate subdirectory.
2. Document the case in that subdirectory’s README.
3. Reference the relevant protocol section (`protocol/0x-*.md`).
4. Open a PR and note any normative implications.

## Implementation requirement

Compatible implementations **SHOULD** pass all published vectors for the protocol versions they claim to support.
