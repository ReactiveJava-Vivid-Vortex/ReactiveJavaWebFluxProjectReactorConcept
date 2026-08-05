# Mono — Topic Overview

## What Is This Topic About? (In Simple Terms)

A `Mono<T>` is Reactor's type for representing **at most one asynchronous value** —
think of it as a reactive version of `Optional<T>` or a `Future<T>`, but lazy and
fully composable with the rest of the reactive world. A `Mono` can settle into
exactly one of **three outcomes**, and nothing else:

1. **Success with a value** — `onNext(value)` then `onComplete()`.
2. **Success with no value (empty)** — just `onComplete()`, like "not found."
3. **Failure** — `onError(throwable)`.

The biggest practical skill in this topic is choosing the right **factory method**
to create a `Mono` for your situation — and the biggest trap is mixing up *eager*
(`Mono.just()`) vs *lazy* (`Mono.fromSupplier()`, `Mono.fromCallable()`,
`Mono.defer()`) creation:

```java
// EAGER — fetchFromDb() runs THE INSTANT this line executes, even with no subscriber!
Mono<User> bad = Mono.just(fetchFromDb());

// LAZY — fetchFromDb() only runs once someone actually subscribes
Mono<User> good = Mono.fromSupplier(() -> fetchFromDb());
```

Since a `Mono` can be empty, you'll constantly reach for `.switchIfEmpty()` or
`.defaultIfEmpty()` to decide what "nothing found" should mean for your specific
case — a fallback value, a different `Mono`, or an error.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Mono.just()** | Wraps an already-known, non-null value — captured **eagerly**, at creation time. |
| 2 | **Mono.empty()** | Completes successfully with no value — represents "nothing to return" without erroring. |
| 3 | **Mono.error()** | Signals failure immediately on subscription — the reactive equivalent of `throw`. |
| 4 | **Mono.fromSupplier()** | Lazily runs a `Supplier<T>` only on subscription; re-runs fresh per subscriber. |
| 5 | **Mono.fromRunnable()** | Runs a side-effecting `Runnable` (no return value) then completes empty — `Mono<Void>`. |
| 6 | **Mono.fromCallable()** | Like `fromSupplier()` but for code that may throw a checked exception. |
| 7 | **Mono.defer()** | Lazily builds a **whole new Mono** per subscriber — use when the decision itself must be fresh. |
| 8 | **Mono.create()** | Manual escape hatch (`MonoSink`) for bridging legacy callback-based APIs into a `Mono`. |
| 9 | **MonoSink** | The "microphone" inside `Mono.create()` — call `success()`, `success(T)`, or `error()` exactly once. |
| 10 | **Mono lifecycle** | Exactly 3 outcomes: value+complete, empty (complete only), or error — never a mix. |
| 11 | **Success vs Empty vs Error** | Treat these as 3 distinct cases in your code — don't conflate "empty" with "error." |
| 12 | **Lazy evaluation** | Nothing in a `Mono` chain runs until subscribed — the same laziness principle as topic 3, applied to `Mono`. |
| 13 | **Subscription** | `.subscribe()` has overloads for value-only, value+error, or value+error+complete handling. |
| 14 | **Logging** | `.log()` shows exactly which of the 3 outcomes occurred and when — great for debugging "why is my Mono empty?" |
| 15 | **Factory methods** | The cheat-sheet-within-a-cheat-sheet: `just` (eager) vs `fromSupplier`/`fromCallable`/`defer` (lazy) vs `empty`/`error` vs `create` (bridging). |

## How It All Fits Together

```
                     Mono<T> created (lazily, usually)
                              │
                    someone calls .subscribe()
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        SUCCESS(value)      EMPTY           ERROR
      onNext + onComplete  onComplete()   onError(throwable)
              │               │               │
      handled by .map()  .switchIfEmpty()  .onErrorResume()
```

Master this one type well, and `Flux` (the next topic) will feel like "the same
ideas, just 0-to-N instead of 0-to-1."
