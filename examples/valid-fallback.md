# Valid Fallback

## Good Code

```ts
const pageSize = query.pageSize === undefined
  ? 20
  : parsePageSize(query.pageSize);

const locale = user.locale ?? "zh-CN";

const sortOrder = query.sortOrder ?? "created_at_desc";
```

## Why It Is Valid

These defaults express product rules:

- missing page size uses the default page size,
- missing locale uses the product default language,
- missing sort order uses the default list order.

They cover absence, not invalid input. If the user passes `pageSize=abc` or `pageSize=-1`, the parser should reject it.

## Operational Fallback

```ts
const requestTimeoutMs =
  env.REQUEST_TIMEOUT_MS === undefined
    ? 5000
    : parseTimeoutMs(env.REQUEST_TIMEOUT_MS);
```

This can be valid when timeout has an operational default and the parser validates range and units.

## Degraded Mode

```ts
try {
  const items = await recommendations.list(userId);
  return { kind: "ok", items };
} catch (error) {
  logger.warn({ error, userId }, "recommendations unavailable");
  return { kind: "degraded", items: [] };
}
```

This fallback is acceptable only if recommendations are non-critical and callers can see degradation.

## Review Questions

- Is this fallback a named product or operational policy?
- Does it cover absence only, not invalid input or system failure?
- Is it low-risk in production?
- Is it observable if it handles dependency failure?
- Are missing, invalid, and dependency-failure cases tested separately?
