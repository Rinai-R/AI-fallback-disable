# Wrong Fallback

## Bad AI Code

```ts
const role = user.role || "user";
const price = product.price || 0;
const tenantId = req.headers["x-tenant-id"] || "default";
const stock = inventory[itemId] || 999;
```

## What It Hides

These fallbacks are not just meaningless. They create false business facts.

- Missing role becomes a real permission level.
- Missing price becomes a free product.
- Missing tenant becomes a real tenant.
- Missing inventory becomes enough stock.

This kind of fallback can change authorization, billing, routing, or state transitions.

## Better Semantics

Validate required facts at the boundary:

```ts
const role = requireUserRole(user);
const price = requireMoney(product.price, "product.price");
const tenantId = requireTenantHeader(req);
const stock = await requireInventory(itemId);
```

If a default is a real product rule, name it:

```ts
const sortOrder = query.sortOrder ?? "created_at_desc";
```

`sortOrder` can be a product default. `price = 0` cannot be assumed without an explicit pricing rule.

## Review Questions

- Does the fallback value become a business fact?
- Does it affect identity, permission, money, tenant, inventory, or state?
- Is missing data different from a legitimate zero, empty, or default value?
- Who approved this fallback as a product rule?
- Would failing early be safer than continuing with fake data?
