# ADR 001: Domain-Oriented Modular Monolith

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Proposed portfolio description; confirm against the actual implementation.
- **Context:** Retail workflows span catalog, cart, pricing, checkout, payment, and commissions. Splitting every domain into an independent service can add operational and distributed-consistency costs.
- **Decision:** Organize related responsibilities into domain modules within one deployable backend, using explicit service boundaries.
- **Consequences:** Deployment and local transaction coordination remain comparatively straightforward. Teams must actively prevent circular dependencies and cross-domain data-access shortcuts.
- **Alternatives considered:** A service-per-domain architecture; a single unstructured application layer.
