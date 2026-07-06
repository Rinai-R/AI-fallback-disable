# Swallowed Database Error

## Bad AI Code

```ts
async function listOrders(userId: string): Promise<Order[]> {
  try {
    return await db.orders.findMany({ where: { userId } });
  } catch {
    return [];
  }
}
```

## Why It Hides Failure

An empty list can mean "the user has no orders." It should not also mean "the database is down," "the query timed out," or "the schema changed."

This code converts an infrastructure failure into a valid business result. Callers, logs, metrics, and users lose the ability to tell the difference.

## Better Semantics

```ts
async function listOrders(userId: string): Promise<Order[]> {
  return db.orders.findMany({ where: { userId } });
}
```

If a known recoverable error exists, handle that specific error and keep it observable:

```ts
async function listOrders(userId: string): Promise<Order[]> {
  try {
    return await db.orders.findMany({ where: { userId } });
  } catch (error) {
    throw new OrderRepositoryError("Failed to list orders", { cause: error });
  }
}
```

## Review Questions

- Is `[]` a valid business result, a dependency failure, or both?
- If the database is down, who can observe it?
- Should this function return a typed result, throw a repository error, or trigger an explicit degraded mode?
- Do tests cover both "no orders" and "database failure" separately?
