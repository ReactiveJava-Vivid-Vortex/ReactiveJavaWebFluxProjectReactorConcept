# Combining Publishers — Topic Overview

## What Is This Topic About? (In Simple Terms)

Real applications rarely work with just one stream at a time — you often need to
call two services in parallel and combine their results, or chain several sources
one after another. This topic is your toolkit for combining multiple
`Mono`/`Flux` sources, and the single most important thing to internalize is the
difference between **sequential** and **concurrent** combination:

- **`concat()`** — subscribes to sources one at a time, in strict order; the second
  source doesn't even start until the first fully completes.
- **`merge()`** — subscribes to all sources at once (concurrently); items are
  emitted in whatever order they actually arrive, interleaved.
- **`zip()`** — subscribes to all sources concurrently, but pairs up their Nth items
  together, stopping at the shortest source.

```java
// merge(): both calls happen IN PARALLEL — much faster than one after another
Mono<UserProfile> profile = userService.getProfile(userId);
Mono<List<Order>> orders = orderService.getOrders(userId);

Mono.zip(profile, orders)
    .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()))
    .subscribe(dashboard -> render(dashboard));
```

A second recurring theme is **error tolerance**: plain `concat()`/`merge()` stop
immediately the moment any source errors, but their `*DelayError` variants
(`concatDelayError()`, `mergeDelayError()`) let every source finish first and only
surface the error at the very end — useful for batch operations where one failure
shouldn't block everything else.

Finally, `firstWithSignal()`/`firstWithValue()` implement a "race" pattern — call
multiple redundant sources and take whichever responds first, cancelling the rest.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **startWith()** | Prepend one or more values (or another Publisher) before the main sequence starts. |
| 2 | **concat()** | Combine sources sequentially — fully exhaust the first before subscribing to the next. |
| 3 | **concatWith()** | Instance-method version of `concat()`, for fluent chaining. |
| 4 | **concatDelayError()** | Like `concat()`, but delays any error until all sources have had a chance to run. |
| 5 | **merge()** | Combine sources concurrently — items interleave based on actual timing, not source order. |
| 6 | **mergeSequential()** | Subscribes concurrently (like `merge`) but emits results in source order (like `concat`). |
| 7 | **mergeDelayError()** | Like `merge()`, but delays any error until all sources have completed. |
| 8 | **zip()** | Pair up items positionally across sources; stops as soon as the shortest source completes. |
| 9 | **zipWith()** | Instance-method version of `zip()`, for fluent chaining — great for parallel service calls. |
| 10 | **combineLatest()** | Recombine using the *latest* value from each source whenever ANY source emits — not strict pairing like `zip()`. |
| 11 | **firstWithSignal()** | Race multiple sources; take whichever emits ANY signal (value/error/complete) first, cancel the rest. |
| 12 | **firstWithValue()** | Like `firstWithSignal()`, but specifically waits for a real *value* — ignores sources that complete empty. |
| 13 | **Practical Use Cases** | Cache+DB fallback (`switchIfEmpty`), parallel calls (`zip`), aggregating microservices (`merge`). |

## How It All Fits Together

```
Do sources need to run ONE AFTER ANOTHER, in strict order?
   │
   ├── YES ──▶ concat() / concatWith()  (+ concatDelayError() if failures shouldn't stop the rest)
   │
   └── NO, they can run AT THE SAME TIME
              │
              ├── Need to PAIR UP results 1-to-1?        ──▶ zip() / zipWith()
              ├── Just want everything, order doesn't matter? ──▶ merge() / mergeSequential()
              ├── Need "latest value from each" recombined?   ──▶ combineLatest()
              └── Want to RACE sources, take the fastest?     ──▶ firstWithSignal() / firstWithValue()
```

Whenever you're about to write two sequential blocking-style calls in a reactive
pipeline, stop and ask: "could these actually run concurrently?" — if yes, `zip()`
or `merge()` will often cut your total latency roughly in half.
