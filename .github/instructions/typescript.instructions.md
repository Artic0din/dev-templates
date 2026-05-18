---
applyTo: "**/*.{ts,tsx}"
---

# TypeScript rules

## Type strictness
- No `any`. Use `unknown` and narrow.
- No `as` casts that lie about types. Use type guards instead.
- Function signatures fully typed (params + return).
- `strict: true` in `tsconfig.json`. `noUncheckedIndexedAccess: true`.

## Next.js (App Router)
- Server components by default. `'use client'` only when needed (event
  handlers, browser APIs, state).
- Server components don't import client-only modules.
- Client components don't import server-only modules (auth secrets, DB).
- Use `Suspense` + `loading.tsx` for streaming.
- Error boundaries via `error.tsx`.

## API routes / route handlers
- Validate input with Zod (or equivalent).
- Return correct HTTP status codes.
- No PII or secrets in response bodies.
- Handle promise rejections — no unhandled rejections.

## Prisma / Drizzle
- Migrations additive only (no destructive schema changes without
  explicit comment in the migration).
- Indexes on foreign keys.
- No raw SQL without parameterisation.

## React
- No new object/array literals in JSX props if they cause re-renders —
  memoise with `useMemo` / `useCallback`.
- `key` prop on every list element (stable, unique).
- No `useEffect` for derived state — derive in render or `useMemo`.

## AEST date bug
Never `toISOString().split('T')[0]` for local dates. Use local components:
```ts
const y = d.getFullYear();
const m = String(d.getMonth() + 1).padStart(2, '0');
const day = String(d.getDate()).padStart(2, '0');
return `${y}-${m}-${day}`;
```
