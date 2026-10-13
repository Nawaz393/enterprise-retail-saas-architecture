# Security and Publication Notes

[← Main README](../README.md) · [← Documentation index](README.md)

Before publishing, ensure every example is approved for disclosure and contains no real secrets, customer data, or confidential business rules.

- [ ] Verify project and personal contribution claims.
- [ ] Remove credentials, tokens, private keys, `.env` files, and database dumps.
- [ ] Use synthetic data in examples and screenshots.
- [ ] Review Git history for accidentally committed secrets.
- [ ] Verify deployment and production claims.

Accuracy notes:

- Query scoping is not equivalent by itself to PostgreSQL-native RLS.
- A PostgreSQL transaction cannot roll back an external payment-provider action.
- A partial unique index may enforce “at most one” without enforcing “at least one.”
- Payment tokenization does not by itself establish PCI-DSS compliance.
