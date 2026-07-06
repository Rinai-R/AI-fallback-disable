# Typed Business Absence

## Bad AI Code

```ts
async function getProfile(userId: string): Promise<Profile | null> {
  try {
    return await profileService.getByUserId(userId);
  } catch {
    return null;
  }
}
```

## Why It Hides Failure

`null` might mean:

- the profile does not exist,
- the profile service is down,
- the request timed out,
- the payload could not be parsed,
- the caller passed invalid state.

Those are different outcomes. Treating all of them as `null` forces every caller to guess.

## Better Semantics

Use a typed result for expected business absence and propagate unexpected failures:

```ts
type ProfileResult =
  | { type: "found"; profile: Profile }
  | { type: "not_found" };

async function getProfile(userId: string): Promise<ProfileResult> {
  const profile = await profileRepository.findByUserId(userId);

  if (!profile) {
    return { type: "not_found" };
  }

  return { type: "found", profile };
}
```

## Review Questions

- Is absence expected business behavior or a system failure?
- Does the return type let the caller distinguish not found from dependency failure?
- Should validation errors, auth errors, and missing data have separate paths?
- Do tests assert each outcome explicitly?
