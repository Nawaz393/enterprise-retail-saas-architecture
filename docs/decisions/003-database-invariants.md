# ADR 003: Database-Level Invariants

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Verify which constraints and triggers are implemented.
- **Context:** Application validators alone may not protect all write paths or concurrent operations.
- **Decision:** Enforce suitable invariants with database constraints, indexes, and selected triggers.
- **Benefits:** Data integrity is protected at the persistence boundary.
- **Trade-offs:** Migrations, trigger tests, bulk-write performance, and troubleshooting require care.
- **Alternatives:** Application-only validation; moving all business rules into triggers.
