# ADR 003: Enforce Critical Invariants at the Database Boundary

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Proposed portfolio description; verify which constraints and triggers are actually implemented.
- **Context:** Application-level validation can be bypassed by alternate write paths or fail under concurrent operations.
- **Decision:** Use appropriate database constraints, indexes, and selected triggers to enforce invariants that must hold across write paths.
- **Consequences:** Data integrity is stronger at the persistence boundary, while migrations, trigger testing, bulk-write performance, and operational troubleshooting require care.
- **Alternatives considered:** Relying only on application validation; enforcing all business rules in triggers.
