# Reactor Operators — Topic Overview

## What Is This Topic About? (In Simple Terms)

If `Mono`/`Flux` are the "pipes," **operators** are everything you can attach to a
pipe to reshape what flows through it. This is the biggest topic in the whole
course because Reactor ships hundreds of operators — but nearly all of them fall
into just seven families, and once you recognize the family, the specific operator
name is easy to guess.

Think of a factory assembly line: each operator is a station the data passes
through, one item at a time, in order:

```java
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)   // Filtering: keep only even numbers
    .map(n -> n * n)            // Transformation: square each one
    .doOnNext(n -> log.info("about to emit {}", n)) // Side-effect: just observe
    .subscribe(System.out::println);
```

The seven families, in the order they appear in this folder:

1. **Transformation** — reshape each item (`map`, `cast`, `index`, `handle`).
2. **Filtering** — decide which items pass through (`filter`, `take`, `skip`, ...).
3. **Default values** — fill in a fallback when the stream is empty (`defaultIfEmpty`,
   `switchIfEmpty`).
4. **Side-effect operators** — observe without changing the data (`doOnNext`,
   `doOnError`, `doFinally`, ...).
5. **Collecting** — gather many items into one collection (`collectList`,
   `collectMap`, ...).
6. **Aggregation** — reduce many items into one summary value (`count`, `reduce`,
   `scan`).
7. **Utility operators** — debugging helpers (`log`, `checkpoint`).

## Quick Revision Cheat Sheet

### 1. Transformation
| Operator | One-Line Summary |
|---|---|
| `map()` | Synchronously transform each item, one-to-one. |
| `cast()` | Change the declared type at runtime (like a Java cast). |
| `index()` | Pair each item with its zero-based position (like `enumerate()`). |
| `handle()` | Combined map+filter+error in one step: transform, skip, or fail per item. |

### 2. Filtering
| Operator | One-Line Summary |
|---|---|
| `filter()` | Keep only items matching a predicate. |
| `take(n)` | Keep only the first `n` items, then cancel upstream. |
| `takeWhile()` | Keep items while a condition is true; stop **before** the first failing item. |
| `takeUntil()` | Keep items until a condition is true; stop **after** (includes) the matching item. |
| `skip(n)` | Discard the first `n` items, keep the rest. |
| `distinct()` | Remove duplicate items (by equality or a custom key). |

### 3. Default Values
| Operator | One-Line Summary |
|---|---|
| `defaultIfEmpty()` | Emit one static fallback value if the source completes empty. |
| `switchIfEmpty()` | Switch to a whole different (possibly async) Mono/Flux if empty. |

### 4. Side-Effect Operators
| Operator | One-Line Summary |
|---|---|
| `doFirst()` | Runs before subscription even begins — earliest hook. |
| `doOnNext()` | Observe each item without changing it (logging, metrics). |
| `doOnSubscribe()` | Runs when a new subscription is established. |
| `doOnRequest()` | Observe how much demand (`request(n)`) flows through. |
| `doOnError()` | Observe an error passing through — doesn't handle/recover it. |
| `doOnComplete()` | Runs only on successful completion, never on error/cancel. |
| `doFinally()` | Runs on ANY ending — success, error, or cancellation (like `finally`). |
| `doOnTerminate()` | Runs on success or error, but NOT on cancellation. |

### 5. Collecting
| Operator | One-Line Summary |
|---|---|
| `collectList()` | Gather all items into a single `Mono<List<T>>`. |
| `collectMap()` | Gather items into a `Mono<Map<K,V>>` using key/value extractor functions. |
| `collectSortedList()` | Like `collectList()` but sorted (natural order or custom `Comparator`). |

### 6. Aggregation
| Operator | One-Line Summary |
|---|---|
| `count()` | Emit the total number of items as a `Mono<Long>`. |
| `reduce()` | Combine all items into one final value (only the final result is emitted). |
| `scan()` | Like `reduce()`, but emits every intermediate running result, not just the final one. |

### 7. Utility Operators
| Operator | One-Line Summary |
|---|---|
| `log()` | Print every Reactive Streams signal passing through this point — best debugging tool. |
| `checkpoint()` | Tag a pipeline location so error stack traces point back to a meaningful spot. |

## How It All Fits Together

```
Flux/Mono source
     │
     ▼  Transformation  (map, cast, index, handle)
     ▼  Filtering       (filter, take, skip, distinct...)
     ▼  Default values  (defaultIfEmpty, switchIfEmpty)
     ▼  Side-effects    (doOnNext, doOnError, doFinally...)  ← observe, don't alter
     ▼  Collecting      (collectList, collectMap...)          ← many → one collection
     ▼  Aggregation     (count, reduce, scan)                 ← many → one summary
     ▼  Utility         (log, checkpoint)                      ← debugging aids
subscribe()
```

Don't try to memorize all 28 at once — recognize which *family* a problem belongs
to ("I need to reshape data" → Transformation; "I need a running total" →
Aggregation), and the right operator becomes obvious.
