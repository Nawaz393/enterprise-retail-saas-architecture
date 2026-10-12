# Security, Privacy, and Public Disclosure

[← Main README](../README.md) · [← Documentation index](README.md)

This repository is designed to demonstrate engineering decisions without disclosing a private application's source or sensitive operational information.

## Before publishing

- [ ] Confirm that the client/employer permits a public case study.
- [ ] Publish only responsibilities you personally performed.
- [ ] Remove credentials, tokens, private keys, `.env` files, database dumps, and production logs.
- [ ] Remove customer names, merchant identifiers, real orders, payment identifiers, and private URLs.
- [ ] Use fabricated data in screenshots, examples, and tests.
- [ ] Check commit history for secrets and confidential files.
- [ ] Verify every technology, scale, security, and deployment claim.
- [ ] Remove or label features that are planned, experimental, or not deployed.
- [ ] Ensure diagrams do not reveal exploitable internal details.

## Avoid unsupported claims

Do not claim “PCI-DSS compliant,” “zero PCI scope,” “production-proven,” “multi-AZ deployed,” “zero public exposure,” or a specific line count unless you have evidence and authorization to publish it. Payment tokenization may reduce exposure to card data but does not itself establish a compliance status.

Application query scoping should not be described as PostgreSQL-native Row-Level Security unless database RLS policies are configured and verified.

## Public/private split

**Public:** sanitized architecture diagrams, high-level component roles, synthetic examples, general trade-offs, and tests that do not expose proprietary rules.

**Private:** application source, credentials, production schemas, customer data, detailed infrastructure configuration, private endpoints, and confidential business workflows.
