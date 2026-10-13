# ADR 002: Decimal Arithmetic for Money

[← ADR index](README.md) · [← Main README](../../README.md)

- **Status:** Confirm the configured library and rounding policy.
- **Context:** Binary floating-point can introduce unexpected representation and rounding behavior.
- **Decision:** Use decimal arithmetic for intermediate monetary calculations and define explicit rounding/reconciliation rules.
- **Benefits:** More predictable precision and visible rounding behavior.
- **Trade-offs:** Currency scale, tax rules, refunds, and remainder allocation still need domain decisions and tests.
- **Alternatives:** Uncontrolled floating-point arithmetic; integer minor units where suitable.
