# Secret Trim

## Bad AI Code

```ts
const jwtSecret = process.env.JWT_SECRET?.trim() || "secret";
const privateKey = process.env.PRIVATE_KEY?.trim();
const webhookSecret = process.env.WEBHOOK_SECRET?.trim() || "";
```

## What It Hides

Secrets are exact-match values. Trimming them can change the value being used for signing, verification, encryption, or authentication.

If leading or trailing whitespace is not allowed, the loader should reject it. It should not silently repair it.

## Better Semantics

```ts
function requireSecret(env: NodeJS.ProcessEnv, name: string): string {
  const value = env[name];

  if (value === undefined || value.length === 0) {
    throw new Error(`${name} is required`);
  }

  if (value !== value.trim()) {
    throw new Error(`${name} must not contain leading or trailing whitespace`);
  }

  return value;
}
```

## Review Questions

- Is this value a secret, token, private key, signature, or exact-match credential?
- Is the code silently changing the value?
- Does the error message avoid printing the secret itself?
- Is there any default secret?
- Is whitespace behavior tested?
