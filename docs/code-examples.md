# Representative Code Examples

[← Main README](../README.md) · [← Documentation index](README.md)

These examples illustrate the architecture described in the case study. They are representative snippets, not a drop-in replacement for the private application. Adapt imports and concrete model types to the actual codebase.

## 1. Fail-closed query scoping

```ts
type ViewScope =
  | { kind: 'all' }
  | { kind: 'organization'; organizationId: number }
  | { kind: 'self'; userId: number }
  | { kind: 'none' }

function applyMerchantScope(query: any, scope: ViewScope) {
  switch (scope.kind) {
    case 'all':
      return query
    case 'organization':
      return query.where('organization_id', scope.organizationId)
    case 'self':
      return query.where('user_id', scope.userId)
    case 'none':
      return query.whereRaw('1 = 0')
  }
}
```

Use the concrete Lucid query-builder type in production. Action permission checks are separate; query scoping alone does not authorize writes.

## 2. Decimal arithmetic

```ts
import Decimal from 'decimal.js'

Decimal.set({
  precision: 20,
  rounding: Decimal.ROUND_HALF_UP,
})

export function calculateTax(
  taxableAmount: string,
  ratePercent: string,
): string {
  const amount = new Decimal(taxableAmount)
  const rate = new Decimal(ratePercent).div(100)

  return amount
    .mul(rate)
    .toDecimalPlaces(2, Decimal.ROUND_HALF_UP)
    .toFixed(2)
}
```

Using decimal strings at the boundary helps avoid importing a pre-rounded JavaScript `number`. Currency scale and tax policy still need to be specified.

## 3. Transactional inventory reservation

```ts
import db from '@adonisjs/lucid/services/db'
import ProductItem from '#product/models/product_item_model'

export async function reserveStock(
  productItemId: number,
  quantity: number,
) {
  if (!Number.isInteger(quantity) || quantity <= 0) {
    throw new Error('Quantity must be a positive integer')
  }

  return db.transaction(async (trx) => {
    const item = await ProductItem.query({ client: trx })
      .where('id', productItemId)
      .forUpdate()
      .firstOrFail()

    if (item.stockQty < quantity) {
      throw new Error('Insufficient stock')
    }

    item.stockQty -= quantity
    await item.save()

    return { productItemId, remainingStock: item.stockQty }
  })
}
```

The stock check and decrement occur while the row is locked. A complete checkout should propagate the same transaction client through all related database writes.

## 4. Database constraints

```sql
ALTER TABLE product_image
ADD CONSTRAINT product_image_exactly_one_owner
CHECK (
  num_nonnulls(product_id, product_item_id, bundle_id) = 1
);

CREATE UNIQUE INDEX product_item_one_active_default
ON product_item (product_id)
WHERE is_default AND NOT is_deleted;
```

The partial unique index enforces **at most one** matching default row. A separate mechanism is needed if the business rule requires exactly one default whenever active variants exist.

## 5. Audit trigger pattern

```sql
CREATE OR REPLACE FUNCTION audit_product_change()
RETURNS trigger AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO audit_log
      (table_name, record_id, action, old_data, new_data, created_at)
    VALUES
      (TG_TABLE_NAME, NEW.id, 'insert', NULL, to_jsonb(NEW), NOW());
    RETURN NEW;
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO audit_log
      (table_name, record_id, action, old_data, new_data, created_at)
    VALUES
      (TG_TABLE_NAME, NEW.id, 'update', to_jsonb(OLD), to_jsonb(NEW), NOW());
    RETURN NEW;
  ELSIF TG_OP = 'DELETE' THEN
    INSERT INTO audit_log
      (table_name, record_id, action, old_data, new_data, created_at)
    VALUES
      (TG_TABLE_NAME, OLD.id, 'delete', to_jsonb(OLD), NULL, NOW());
    RETURN OLD;
  END IF;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

Adapt the audit schema, actor attribution, redaction, and retention policy to the actual database.

## Test cases worth documenting

- A user cannot access records outside their organization scope.
- Missing scope relationships fail closed.
- Concurrent purchases cannot oversell the final stock unit.
- Payment failure does not leave an incorrectly finalized local order.
- Decimal calculations match rounding expectations at boundary values.
- Invalid ownership combinations are rejected by PostgreSQL.
- Retried jobs do not duplicate external side effects.

Next: [Authorization](authorization.md) · [Checkout](checkout-and-inventory.md) · [Pricing](pricing-and-tax.md) · [Database integrity](database-integrity.md)
