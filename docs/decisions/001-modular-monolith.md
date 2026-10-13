# ADR 001: Domain-Oriented Modular Monolith

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Confirm against the actual implementation.
- **Context:** Retail workflows span catalog, cart, pricing, orders, payments, and commissions.
- **Decision:** Organize capabilities into domain modules within one backend, using explicit service boundaries.
- **Benefits:** Straightforward deployment and shared database transactions where appropriate.
- **Trade-offs:** Boundaries need discipline to prevent circular dependencies and cross-domain shortcuts.
- **Alternatives:** Independent services per domain; one unstructured service layer.
