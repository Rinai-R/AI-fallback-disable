# Swallowed Error Fallback

## Bad AI Code

```ts
async function getUser(id: string) {
  try {
    return await db.users.findById(id);
  } catch {
    return null;
  }
}

async function getBalance(userId: string) {
  try {
    return await paymentClient.getBalance(userId);
  } catch {
    return 0;
  }
}

async function canEdit(userId: string, docId: string) {
  try {
    return await permissionService.canEdit(userId, docId);
  } catch {
    return false;
  }
}
```

## What It Hides

The code collapses system failure into normal business results.

- Database failure becomes user not found.
- Payment service failure becomes zero balance.
- Permission service failure becomes ordinary denial.

Returning the safest-looking value is still wrong when the caller needs to know the system is unhealthy.

## Better Semantics

Separate business absence from dependency failure:

```ts
type UserResult =
  | { kind: "found"; user: User }
  | { kind: "not_found" };

async function getUser(id: string): Promise<UserResult> {
  const user = await db.users.findById(id);
  return user ? { kind: "found", user } : { kind: "not_found" };
}
```

Let unexpected dependency errors propagate, or map them to an explicit system error at the boundary:

```ts
try {
  return await getUser(id);
} catch (error) {
  logger.error({ error, id }, "user lookup failed");
  throw new ServiceUnavailableError("User lookup failed");
}
```

## Valid Degradation

Some non-core features can degrade:

```ts
try {
  return { kind: "ok", items: await recommendations.list(userId) };
} catch (error) {
  logger.warn({ error, userId }, "recommendations degraded");
  metrics.increment("recommendations.degraded");
  return { kind: "degraded", items: [] };
}
```

The result says degradation happened. It is not disguised as "there are no recommendations".

## Review Questions

- Is the caught error expected business absence or unexpected system failure?
- Can the caller distinguish not found from dependency failure?
- Will this fallback be cached, stored, billed, authorized, or shown as a fact?
- Is the fallback observable through logs, metrics, status, or typed result?
- Should this be propagated instead of converted to empty data?
