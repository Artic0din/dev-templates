---
applyTo: "**/*.swift"
---

# Swift rules

## Concurrency
- `Sendable` correctness — no data races. Compiler warnings off = bugs.
- `@MainActor` on UI code. Don't touch UI from background actors.
- `async let` for concurrent independent work; `TaskGroup` for dynamic.
- Cancellation: check `Task.isCancelled` in long loops.
- No `unsafe` continuations without a comment explaining why.

## Memory
- `[weak self]` in closures that outlive the enclosing scope (network
  callbacks, Combine sinks, NotificationCenter observers).
- Avoid retain cycles in delegate patterns — `weak var delegate`.

## SwiftUI
- `@State` for local view state, `@Bindable` for owned model state.
- `@Observable` macro over `ObservableObject` (iOS 17+).
- No work in view body — compute in `task {}`, `onAppear`, or model.
- `Equatable` conformance for view models if recomposition matters.

## Error handling
- `throws` over `Result` for synchronous errors.
- `Result` for callbacks where async/await isn't available.
- No force unwraps (`!`) without a comment justifying them.
- `try?` only when nil-is-fine; otherwise `try` + handle.

## Date and time
- `Calendar.current` for user-facing dates.
- Store dates as `Date` in DB / model; convert to `DateComponents` for
  display.
- `ISO8601DateFormatter` for wire format; `DateFormatter` for display.
