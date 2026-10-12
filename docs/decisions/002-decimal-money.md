# ADR 002: Decimal Arithmetic for Money

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Proposed portfolio description; confirm library and rounding configuration.
- **Context:** Binary floating-point representation can produce unexpected precision behavior for monetary calculations.
- **Decision:** Use decimal arithmetic for intermediate financial calculations and define explicit rounding and reconciliation rules.
- **Consequences:** Calculations become more predictable, but business rules still need to specify currency scale, tax treatment, rounding points, refunds, and remainder allocation.
- **Alternatives considered:** Uncontrolled binary floating-point arithmetic; integer minor-unit arithmetic where appropriate.
