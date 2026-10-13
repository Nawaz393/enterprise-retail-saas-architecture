# Pricing, Discounts, and Tax

[← Main README](../README.md) · [← Documentation index](README.md) · [Code examples](code-examples.md)

```mermaid
flowchart TD
    ITEMS[Cart items and prices] --> SUB[Calculate subtotal]
    SUB --> DISC[Allocate discounts]
    DISC --> BASE[Determine taxable amount]
    BASE --> TAX[Apply applicable tax rules]
    TAX --> ROUND[Apply currency rounding]
    ROUND --> CHECK{Totals reconcile?}
    CHECK -->|No| DELTA[Apply documented remainder rule]
    CHECK -->|Yes| SAVE[Persist totals and snapshots]
    DELTA --> SAVE
```

## Bundle discount allocation

A proportional allocation can use each component's share of the bundle's list price:

$$
w_i = \frac{p_i}{\sum_{j} p_j}
$$

$$
d_i = D \times w_i
$$

Here, \(p_i\) is the component's list-price contribution and \(D\) is the total discount. The implementation must define rounding and how a remaining minor-unit difference is assigned.

## Review checklist

- Define the rounding mode and when rounding occurs.
- Reconcile item totals against the order total.
- Store a snapshot of prices, discounts, and taxes used at checkout.
- Test boundary values, refunds, and partial returns.
- Validate tax rules against applicable business requirements.

See the [Decimal.js example](code-examples.md#2-decimal-arithmetic). Decimal arithmetic improves numerical control but does not by itself prove accounting or tax compliance.
