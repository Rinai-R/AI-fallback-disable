# Semantic Confusion

## Bad AI Code

```ts
async function loadDocument(userId: string, docId: string) {
  try {
    const doc = await db.docs.findById(docId);
    if (!doc) return null;
    if (doc.ownerId !== userId) return null;
    return doc;
  } catch {
    return null;
  }
}
```

## What It Hides

`null` now means too many things:

- document not found,
- user is forbidden,
- database failed,
- invalid or corrupt document state.

The caller cannot pick the right response, log severity, retry behavior, or user message.

## Better Semantics

Return explicit business states and keep system errors visible:

```ts
type LoadDocumentResult =
  | { kind: "found"; doc: Document }
  | { kind: "not_found" }
  | { kind: "forbidden" };

async function loadDocument(userId: string, docId: string): Promise<LoadDocumentResult> {
  const doc = await db.docs.findById(docId);
  if (!doc) return { kind: "not_found" };
  if (doc.ownerId !== userId) return { kind: "forbidden" };
  return { kind: "found", doc };
}
```

Unexpected database errors should not be returned as `not_found`.

## Review Questions

- How many states does this one fallback value represent?
- Does the caller need different HTTP status codes, retries, logs, or messages?
- Is empty result a valid business state or a hidden system failure?
- Does the existing contract call for a typed union or domain error? Use an explicit degraded result only when a pre-existing interface defines it; do not invent one during cleanup.
