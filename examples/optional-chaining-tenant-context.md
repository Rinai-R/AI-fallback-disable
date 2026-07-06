# Optional Chaining In Tenant Context

## Bad AI Code

```ts
async function listProjectFiles(ctx: RequestContext, projectId: string) {
  const tenantId = ctx.user?.tenant?.id;

  return db.files.findMany({
    where: {
      projectId,
      tenantId,
    },
  });
}
```

## Why It Hides Failure

`tenantId` is not an optional display field here. It is part of the authorization and isolation boundary.

If the tenant context is missing, continuing the query can produce one of several bad outcomes depending on the ORM and schema: invalid query, empty result, broad query, or a confusing runtime error. None of those states clearly says "the authenticated tenant contract was broken."

## Better Semantics

Validate identity and tenant context before core logic receives it:

```ts
type AuthenticatedContext = {
  userId: string;
  tenantId: string;
};

async function listProjectFiles(ctx: AuthenticatedContext, projectId: string) {
  return db.files.findMany({
    where: {
      projectId,
      tenantId: ctx.tenantId,
    },
  });
}
```

If the request has no tenant, reject it at the boundary with an auth or tenant-context error.

## Review Questions

- Is this field genuinely optional, or is it an invariant after authentication?
- What should happen when user, tenant, or permission context is missing?
- Are unauthorized, forbidden, not found, and empty result represented differently?
- Do queries, cache keys, events, and logs preserve tenant boundaries?
