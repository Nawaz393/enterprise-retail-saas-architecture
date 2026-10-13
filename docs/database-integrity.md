# Database Integrity and Audit History

[← Main README](../README.md) · [← Documentation index](README.md) · [Code examples](code-examples.md)

## Patterns

- Partial unique indexes for conditional uniqueness.
- `CHECK` constraints for row-level invariants.
- Foreign keys for referential integrity.
- Deferred constraint triggers for cross-row rules evaluated at commit.
- Audit triggers for selected before/after snapshots.

See [SQL examples](code-examples.md#4-database-constraints) and the [audit trigger example](code-examples.md#5-audit-trigger-pattern).

```mermaid
flowchart LR
    MUTATION[Insert / Update / Delete] --> RULES[Constraints and indexes]
    RULES --> TRIGGER[Relevant trigger checks]
    TRIGGER --> VALID{Valid?}
    VALID -->|Yes| COMMIT[Commit]
    VALID -->|No| ROLLBACK[Reject transaction]
    MUTATION -. selected changes .-> AUDIT[Audit event]
    AUDIT --> HISTORY[(Audit history)]
```

Test deferred constraints, migrations, bulk writes, trigger performance, retention, and audit access controls. Audit payloads should not store credentials, payment data, or unnecessary personal information.
