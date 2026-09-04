# Security Policy

## Supported Versions

HCPP is currently in **v0.1 Draft / Experimental** status. Security fixes will be applied to the latest draft documentation, schemas, and any published reference code on a best-effort basis.

| Version | Supported |
|---------|-----------|
| 0.1 Draft | Yes (best effort) |
| < 0.1 | No |

Once a stable v1.0 is released, this policy will be updated with clearer support windows.

## Reporting a Vulnerability

We take security issues seriously.

**Please do not report security vulnerabilities through public GitHub Issues.**

Instead, please report them via one of the following channels:

1. **GitHub Private Vulnerability Reporting** 
   Use the “Security” tab → “Report a vulnerability”.

2. **Email**  
   Send details to: `fushou@hxxbot.com`

### What to include

To help us understand and address the issue quickly, please include:

- Description of the vulnerability
- Affected component (protocol specification, schema, reference implementation, SDK, etc.)
- Steps to reproduce
- Potential impact
- Any suggested fix (optional)
- Your contact information (optional, for follow-up)

### Response expectations

- We will acknowledge receipt within **72 hours** (best effort during the experimental phase).
- We will provide an initial assessment and expected timeline as soon as practical.
- We ask that you give us a reasonable time to investigate and mitigate before any public disclosure.

## Scope

In scope examples:

- Cryptographic design or specification flaws that undermine integrity or authenticity guarantees
- Issues in reference implementations or official SDKs that could lead to signature bypass, hash mismatch acceptance, key misuse, etc.
- Practical attacks against the stated trust model (within documented assumptions)

Out of scope examples:

- Issues only present in unmodified third-party dependencies (please report upstream)
- Social engineering, physical attacks
- Denial of service with unrealistic resource assumptions
- Bugs in unofficial third-party implementations (unless they reveal a protocol-level problem)

## Security Considerations

Implementers and operators should also read:

- [docs/security-considerations.md](docs/security-considerations.md)
- [docs/threat-model.md](docs/threat-model.md)
- [docs/trust-model.md](docs/trust-model.md)

Key reminders:

- Publisher private keys must never be uploaded to the Registry.
- Always validate both cryptographic results **and** status (e.g. revoked / superseded).
- Follow the canonicalization and hashing rules exactly to avoid interoperability and security surprises.

## Acknowledgments

We appreciate responsible disclosure and will credit reporters who wish to be acknowledged (unless they prefer to remain anonymous).

---

Thank you for helping keep HCPP and its users safe.
