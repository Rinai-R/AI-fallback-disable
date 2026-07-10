# Valid Fallback

## Evidence-Backed Existing Defaults

Assume the repository already contains these policies:

- an API contract test asserts that omitted `pageSize` means `20`;
- the product localization policy defines `zh-CN` for users with no locale;
- the query contract defines `created_at_desc` when sort order is omitted.

Given that evidence, keep the existing defaults:

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

Do not infer these rules from the values themselves. "Common," "low-risk," or "reasonable" is not evidence, and a cleanup task should not add equivalent defaults to neighboring fields.

## Operational Fallback

Assume an existing operations policy defines a 5000 ms timeout when the variable is absent. Preserve that default while continuing to reject blank, malformed, and out-of-range input:

```ts
const requestTimeoutMs =
  env.REQUEST_TIMEOUT_MS === undefined
    ? 5000
    : parseTimeoutMs(env.REQUEST_TIMEOUT_MS);
```

This is valid because the policy predates the cleanup, not merely because timeouts often have defaults.

## Degraded Mode

Assume an existing interface defines `kind: "degraded"`, callers handle it, and the service runbook approves empty recommendations during a recommendation outage:

```ts
try {
  const items = await recommendations.list(userId);
  return { kind: "ok", items };
} catch (error) {
  logger.warn({ error, userId }, "recommendations unavailable");
  return { kind: "degraded", items: [] };
}
```

Keep this fallback because the existing policy, result contract, and observability prove it is intentional. Do not introduce this `try/catch` during a fallback-removal task merely because recommendations look non-critical.

## Review Questions

- Does cited evidence define this product or operational policy?
- Where is that policy evidenced in the repository or user requirement?
- Does it cover absence only, not invalid input or system failure?
- Is it low-risk in production?
- Is it observable if it handles dependency failure?
- Are missing, invalid, and dependency-failure cases tested separately?
- Did the cleanup leave this valid fallback unchanged instead of duplicating or generalizing it?
