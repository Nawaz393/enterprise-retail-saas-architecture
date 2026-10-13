# Checkout and Inventory Consistency

[← Main README](../README.md) · [← Documentation index](README.md) · [Code examples](code-examples.md)

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant API as Checkout API
    participant DB as PostgreSQL Transaction
    participant Pay as Payment Provider
    Client->>API: Submit checkout
    API->>DB: Begin transaction
    API->>DB: Lock and validate stock
    DB-->>API: Current inventory state
    API->>API: Calculate pricing and tax
    API->>DB: Write order and line items
    API->>Pay: Initiate payment
    alt Payment accepted
        Pay-->>API: Provider result
        API->>DB: Persist payment state
        API->>DB: Commit
        API-->>Client: Confirmation
    else Payment rejected
        Pay-->>API: Decline
        API->>DB: Roll back local changes
        API-->>Client: Failure response
    end
```

A row lock can serialize competing attempts to reserve the same inventory row. See the [transactional stock example](code-examples.md#3-transactional-inventory-reservation).

## External payment caveat

A database rollback cannot undo an external payment-provider side effect. Timeouts can leave the provider outcome unknown, or the provider may approve just before the application fails to persist the result.

A robust workflow should consider idempotency keys, explicit payment states, verified webhooks, retry safety, reconciliation, and inventory release/compensation rules.

## Test scenarios

- Two checkouts compete for the final stock unit.
- The same request is submitted twice.
- Payment is declined or times out.
- Payment succeeds but the local transaction fails to commit.
- A webhook or worker retries an already-processed event.
