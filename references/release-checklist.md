# AI Generated Code Release Checklist

Use this reference when AI-assisted changes touch configuration, secrets, auth, external input, dependency behavior, deployment, CI, or production release paths.

## Configuration

- All environment variables are read in one config module.
- Startup completes parsing, type validation, range validation, and required-field validation.
- Business code depends on typed config, not `process.env`.
- Required config missing, blank, or malformed fails startup.
- Low-risk defaults are limited to operational tuning such as `PORT`, `LOG_LEVEL`, request timeout, page size, or default-off feature flags.
- Config handling distinguishes missing, blank, invalid, and valid falsy values.
- Numbers check integer/float expectations, positivity, bounds, and units.
- Booleans accept only explicit strings such as `true/false` or `1/0`.
- URLs parse with URL tooling and validate protocol, host, and required path.

## Secrets

- Secrets, API keys, database URLs, OAuth client secrets, JWT/session secrets, encryption keys, signing keys, webhook secrets, tenant IDs, and production endpoints have no defaults.
- Missing, blank, or malformed secrets fail startup.
- Secrets are not silently trimmed or case-normalized.
- Logs, errors, traces, metrics, snapshots, and CI output redact secrets.
- Frontend-public config and server secrets have clear naming boundaries.
- Test secrets are isolated to tests and cannot leak into production builds.
- Secret rotation has a plan for dual-key windows, rollback, expiry, and monitoring.
- Secret scanning runs in CI where available.

## Permissions And Identity

- Identity, tenant, and permission context is verified at the boundary.
- Missing identity or tenant never falls back to default user, default tenant, admin, or public access.
- Unauthorized, forbidden, not found, and business-empty outcomes have distinct semantics.
- Queries, caches, events, and audit logs include tenant/resource boundaries where needed.
- Server-side write paths enforce permissions.
- Negative tests cover anonymous access, low-privilege user, cross-tenant access, expired credential, and forged ID.
- Service accounts, CI tokens, and webhook credentials use least privilege and rotation/expiry.

## External Input

- HTTP body, query, headers, CLI args, files, queue messages, and webhook payloads are schema-validated at boundaries.
- Core functions receive typed input rather than raw request objects.
- Invalid input returns explicit validation errors.
- Normalization rules are deliberate; signatures, tokens, idempotency keys, and exact-match IDs are not silently trimmed.
- Date, number, enum, URL, email, and ID values parse with explicit rejection paths.
- Webhooks validate signature, timestamp window, event ID idempotency, and trusted source.
- File upload paths limit size, type, path behavior, and parsed content shape.

## Logging And Observability

- Unexpected failures preserve error context and reach centralized handling or clear propagation.
- System failures create observable signals through structured logs, metrics, traces, or alerts.
- Logs include request/trace IDs and safe user/tenant/resource identifiers where useful.
- Logs redact secrets, tokens, cookies, Authorization headers, private keys, full connection strings, and sensitive PII.
- Expected business results and system exceptions are distinguishable in return values and telemetry.
- Important actions have audit events with actor, action, resource, result, and timestamp.
- Fallbacks, retries, circuit breakers, and degraded modes emit metrics or logs.

## Rollback And Degradation

- The release has a rollback path for code, config, database, cache, queues, feature flags, and external contracts.
- Database migrations are forward/backward compatible or split into safe stages.
- Risky behavior is behind a feature flag or gradual rollout.
- Degradation is an explicit product/operations decision, not a silent code fallback.
- Degraded results are visible to monitoring and, where appropriate, API/UI consumers.
- Dependency failures have explicit policy: fail, retry, circuit-break, queue, or degrade.
- Deployment order accounts for multi-version compatibility.

## Monitoring

- RED/USE basics are monitored: request rate, error rate, duration, saturation/resource usage.
- Key business success rates are monitored.
- Fallback, degradation, retry, circuit-break, and unusual empty-result rates are observable.
- Databases, caches, queues, object storage, and third-party APIs have health metrics.
- Alerts include environment, version, service, region, and investigation links or trace examples.
- Post-release observation compares new and old version metrics.
- Liveness and readiness checks are distinct; readiness covers key initialization or dependencies.

## Tests

- Config tests cover missing, blank, invalid, valid, falsy, and out-of-range values.
- Secret tests cover missing failure, rejected defaults, and redacted output.
- Permission tests cover unauthenticated, unauthorized, cross-tenant, and expired-token cases.
- Input schema tests cover missing fields, invalid types, malicious payloads, and boundary values.
- Error tests assert error type, code, or structured result, not only falsey output.
- Dependency failure tests assert propagation, retry, circuit-break, or explicit degradation behavior.
- Migration/schema changes include compatibility or migration tests when relevant.
- CI tests are deterministic and do not depend on the developer machine or real third-party services.

## CI And Release Gates

- CI runs type checks, lint, unit tests, key integration tests, and build.
- High-risk services run dependency scan, secret scan, container scan, or SAST where available.
- CI does not use production secrets.
- CI blocks accidental `.only`, unjustified skips, suspicious snapshot growth, and fixture secrets.
- Build artifacts are traceable to commit SHA, lockfile, and image digest where relevant.
- Database migrations are validated before production.
- Release notes or deployment checklist include config completeness, feature flag state, monitoring, alert owner, and rollback notes.

## Final Questions

- Is this fallback a product requirement, an operations degradation strategy, or an AI convenience?
- Could this default cause a security, data, permissions, billing, or tenant-isolation incident in production?
- Are missing, invalid, blank, and valid falsy values distinct?
- Does this catch block handle a known recoverable error or swallow an unknown failure?
- Does optional chaining hide required identity, tenant, config, or domain state?
- If a dependency fails, what do the user, API caller, logs, metrics, and alerts see?
- If the release rolls back, are database, queue, cache, and config states still compatible?
- Can CI catch config mistakes, secret leaks, permission bypass, and input pollution before production?
