# Changelog

All notable changes to the HCPP protocol specification, schemas, test vectors, and related documentation will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to the versioning policy described in [VERSIONING.md](VERSIONING.md).

---

## [Unreleased]

### Added
- Initial repository structure
- Apache License 2.0
- README.md
- VERSIONING.md
- Design document (v0.1 Draft)

---

## [0.1.0-draft] - 2026-09-04

### Added
- Protocol version `0.1` (Draft / Experimental)
- Object Identifier specification
  - Format: `HXX-{TYPE}-{UUIDv7}`
  - UUIDv7 must be uppercase and retain hyphens (8-4-4-4-12)
- Content Hash specification
  - Algorithm: SHA-256
  - Text content: strip UTF-8 BOM + normalize newlines to LF
  - Binary content: hash raw bytes
  - No semantic normalization or whitespace folding
- Core design principles (ID/Hash separation, append-only, no full content storage by default, optional blockchain anchor, crypto agility)
- High-level object model (Artifact, Publisher, Revision, Reference, Proof)
- Initial versioning and compatibility policy
- Planned integration points for HxxNewsletter and HXXBOT Skills

### Notes
- This is an **Experimental** draft. Breaking changes may still occur before v0.1 is finalized.
- No stable Registry API or SDK is published yet.
- Test vectors are not yet available.

---

## Version Legend

- **Added** for new features / sections
- **Changed** for changes in existing functionality or normative text
- **Deprecated** for soon-to-be removed features
- **Removed** for now removed features
- **Fixed** for any bug / ambiguity fixes in the specification
- **Security** for security-related changes
