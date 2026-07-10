# Removal-First Refactor

Use direct deletion when types, schemas, or existing boundary code already establish the contract. Do not replace fallback with a new guard, wrapper, helper, or exception layer.

## Already-Validated Core Value

Contract evidence:

- `User.profile.displayName` is required by the validated domain type.
- Missing data is a contract violation owned by the input boundary.

Before:

```ts
interface User {
  profile: {
    displayName: string;
  };
}

export function auditLabel(user: User): string {
  const name = user?.profile?.displayName ?? "Unknown";
  return `actor=${name}`;
}
```

After:

```ts
export function auditLabel(user: User): string {
  return `actor=${user.profile.displayName}`;
}
```

Do not "harden" the result with `if (!user)`, an assertion, a new validator, or `try/catch`. The type and boundary already own that requirement.

## Natural Error Propagation

Contract evidence:

- `repo.listByUser` returns `[]` when there are no orders.
- Dependency failure rejects and the existing application error boundary handles it.

Before:

```ts
export async function listOrders(repo: OrderRepository, userId: string) {
  try {
    return await repo.listByUser(userId);
  } catch (error) {
    logger.warn({ error, userId }, "order lookup failed");
    return [];
  }
}
```

After:

```ts
export async function listOrders(repo: OrderRepository, userId: string) {
  return repo.listByUser(userId);
}
```

Do not replace this with `.catch()`, catch-and-rethrow, a wrapped error, retry, or a different empty value. Natural propagation already has the required semantics.

## Preserve Only the Proven Default

Contract evidence:

- A product contract defines missing `pageSize` as `20`; supplied values are validated upstream.
- The auth schema requires `tenantId`; no default tenant exists.

Before:

```ts
export function buildRequest(query: ParsedQuery, auth: AuthContext) {
  return {
    pageSize: query.pageSize ?? 20,
    tenantId: auth.tenantId || "default",
  };
}
```

After:

```ts
export function buildRequest(query: ParsedQuery, auth: AuthContext) {
  return {
    pageSize: query.pageSize ?? 20,
    tenantId: auth.tenantId,
  };
}
```

The valid product default remains unchanged. The fake tenant fallback is removed without a replacement.

## Fallback Delta

Count fallback behavior, not just syntax:

```text
removed: 3
retained: 1
added: 0
net: -3
```

Moving a removed fallback into a helper, catch, retry, compatibility field, alternate provider, or degraded result would make this refactor fail even if the original syntax disappeared.
