---
applyTo: "**/*.{js,mjs,cjs}"
---

# Node.js (JavaScript) rules

## Module system
- ESM (`"type": "module"` in package.json). No CJS for new code.
- `import` not `require` unless interop forces it.
- Top-level await OK.

## Error handling
- No unhandled promise rejections.
- `try/catch` around every `await` in a top-level handler.
- Don't swallow errors — log + re-throw or propagate.

## Async
- No `for...in` over async work. Use `Promise.all` for parallel,
  `for...of` for serial.
- AbortController for cancellable fetches.

## Security
- No `eval`, no `Function()` constructor with user input.
- `crypto.randomUUID()` for IDs, not `Math.random()`.
- `child_process.exec` with user input is RCE — use `execFile` + args array.

## Dependencies
- `pnpm` for installs. `pnpm-lock.yaml` committed.
- No `npm install` without explicit reason — keep lockfile consistent.
