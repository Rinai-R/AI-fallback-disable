# Dependency Degradation Policy

## Bad AI Code

```ts
async function getRecommendation(userId: string) {
  try {
    return await recommenderClient.getForUser(userId);
  } catch {
    return defaultRecommendations;
  }
}
```

## Why It Hides Failure

Fallback to defaults might be a valid product degradation strategy. It might also hide an outage for hours.

The difference is not the code shape; it is whether the fallback has a policy:

- When is it allowed?
- Is the result marked as degraded?
- Is it logged or counted?
- Does it affect billing, ranking, personalization, or safety?
- Can operators see and stop it?

## Better Semantics

Make degradation explicit:

```ts
async function getRecommendation(userId: string): Promise<RecommendationResult> {
  try {
    return {
      type: "personalized",
      items: await recommenderClient.getForUser(userId),
    };
  } catch (error) {
    metrics.increment("recommendations.degraded");
    logger.warn({ error, userId }, "Using default recommendations");

    return {
      type: "degraded",
      reason: "recommender_unavailable",
      items: defaultRecommendations,
    };
  }
}
```

## Review Questions

- Is this fallback approved product behavior or just an AI convenience?
- Can the caller and UI tell the result is degraded?
- Are logs, metrics, alerts, or dashboards connected to this path?
- Does the degraded result have tests?
- Is returning defaults safer than failing the request for this product surface?
