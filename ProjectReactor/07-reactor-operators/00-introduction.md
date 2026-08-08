# Q1. What Are Reactor Operators?

## Simple Explanation (Think of a Factory Assembly Line)

If `Mono`/`Flux` are conveyor belts, **operators** are the stations bolted onto
them. Each item travels down the belt, passing through station after station, in
order, getting reshaped, filtered, or just observed along the way.

```
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)   // Station 1: keep only even numbers
    .map(n -> n * n)            // Station 2: square each one
    .doOnNext(n -> log(n))      // Station 3: just watch, don't touch
    .subscribe(System.out::println);
```

There are **hundreds** of operators in Reactor, but almost all of them fall into
just **seven families.** Once you recognize which family a problem belongs to, the
right operator name is easy to guess.

---

## Q2. What Are the Seven Families, in One Table?

| # | Family | Job | Example Operators |
|---|---|---|---|
| 1 | Transformation | Reshape each item, one-to-one | `map`, `cast`, `index`, `handle` |
| 2 | Filtering | Decide which items pass through | `filter`, `take`, `takeWhile`, `skip`, `distinct` |
| 3 | Default values | Fill in a fallback when empty | `defaultIfEmpty`, `switchIfEmpty` |
| 4 | Side-effects | Observe WITHOUT changing data | `doOnNext`, `doOnError`, `doFinally` |
| 5 | Collecting | Gather many items into one collection | `collectList`, `collectMap` |
| 6 | Aggregation | Reduce many items into one summary | `count`, `reduce`, `scan` |
| 7 | Utility | Debugging helpers | `log`, `checkpoint` |

---

## Q3. Family 1 — Transformation: How Do I Reshape Data?

```java
Flux.just(1, 2, 3)
    .map(n -> n * 10)              // synchronous, one-to-one
    .cast(Number.class)             // change the declared type
    .index()                        // pair each item with its position: Tuple2<Long, T>
    .subscribe(t -> System.out.println(t.getT1() + ": " + t.getT2()));
```

`handle()` combines map+filter in one step — transform, skip, or error per item:

```java
Flux.just(1, 2, 3, 4, 5, 6)
    .handle((n, sink) -> { if (n % 2 == 0) sink.next(n * 10); }) // odd numbers silently skipped
    .subscribe(v -> System.out.println("Got: " + v)); // 20, 40, 60
```

---

## Q4. Family 2 — Filtering: How Do I Decide What Passes?

```java
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)     // keep matching items
    .take(3)                      // keep first 3, then cancel upstream
    .subscribe(n -> System.out.println("Got: " + n)); // 2, 4, 6
```

`takeWhile` vs `takeUntil` — the classic gotcha:

```java
Flux.just(1, 2, 3, 10, 4, 5)
    .takeWhile(n -> n < 5)   // stops BEFORE the failing item -> 1, 2, 3
    .subscribe(System.out::println);

Flux.just(1, 2, 3, 10, 4, 5)
    .takeUntil(n -> n > 5)   // stops AFTER (includes) the matching item -> 1, 2, 3, 10
    .subscribe(System.out::println);
```

---

## Q5. Family 3 — Default Values: What If the Stream Is Empty?

```java
Mono<String> userName = findUserName("unknown"); // returns Mono.empty()

userName.defaultIfEmpty("Guest")                 // static fallback value
    .subscribe(System.out::println);              // Guest

userName.switchIfEmpty(database.findByAlias(id))  // switch to a WHOLE different Mono
    .subscribe(System.out::println);
```

`switchIfEmpty()` is more powerful — the fallback can itself be asynchronous (e.g.
a secondary database lookup).

---

## Q6. Family 4 — Side-Effect Operators: How Do I Just Observe?

```java
Flux.just(1, 2, 3)
    .doOnSubscribe(s -> System.out.println("Subscribed"))
    .doOnNext(n -> System.out.println("About to process: " + n))
    .doOnComplete(() -> System.out.println("Success!"))
    .doOnError(e -> System.out.println("Logged error: " + e))
    .doFinally(signal -> System.out.println("ALWAYS runs: " + signal)) // success, error, OR cancel
    .subscribe();
```

| Operator | Runs On... |
|---|---|
| `doOnNext` | Every item, without modifying it |
| `doOnComplete` | Success only |
| `doOnError` | Error only (doesn't handle/recover it — just observes) |
| `doOnTerminate` | Success OR error, but NOT cancellation |
| `doFinally` | Success, error, OR cancellation — like a `finally` block |

---

## Q7. Family 5 — Collecting: How Do I Get One List Back?

```java
Flux.just(1, 2, 3)
    .collectList()                                    // Mono<List<Integer>>
    .subscribe(list -> System.out.println(list));      // [1, 2, 3]

Flux.just(order1, order2)
    .collectMap(Order::getId, Order::getTotal)          // Mono<Map<K,V>>
    .subscribe(map -> System.out.println(map));
```

**Gotcha:** `.collectList()` requires the source to actually complete — it will
never emit on a truly infinite `Flux`.

---

## Q8. Family 6 — Aggregation: `count()` vs `reduce()` vs `scan()`

```java
Flux.just(1, 2, 3, 4, 5).count()
    .subscribe(n -> System.out.println("Count: " + n)); // 5

Flux.just(1, 2, 3, 4, 5).reduce((sum, next) -> sum + next)
    .subscribe(sum -> System.out.println("Sum: " + sum)); // 15 (ONLY the final result)

Flux.just(1, 2, 3, 4, 5).scan((sum, next) -> sum + next)
    .subscribe(running -> System.out.println("Running: " + running)); // 1, 3, 6, 10, 15 (EVERY step)
```

**The key distinction:** `reduce()` emits ONE final value; `scan()` emits EVERY
intermediate running value — same math, different granularity.

---

## Q9. Family 7 — Utility: How Do I Debug a Pipeline?

```java
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .checkpoint("division-step")   // tags the pipeline location in stack traces
    .log()                          // prints every signal
    .subscribe();
```

---

## Q10. Interview-Style Q&A

### What's the difference between `map()` and `handle()`?

`map()` transforms every item, always. `handle()` can transform, silently skip
(like a filter), or emit an error — all in one operator.

### Does `doOnNext()` let me modify the item?

**No.** It's purely observational — the item passes through unchanged. Use
`map()` if you need to change it.

### If a `Flux` never completes, will `.collectList()` ever emit?

**No** — it waits for `onComplete()`, which never arrives on a truly infinite
stream. This is a very common source of "why is my code just hanging?" bugs.

---

## Q11. Summary

```
Flux/Mono source
     │
     ▼  Transformation   (map, cast, index, handle)         — reshape each item
     ▼  Filtering        (filter, take, skip, distinct)      — decide what passes
     ▼  Default values   (defaultIfEmpty, switchIfEmpty)     — fill in for empty
     ▼  Side-effects     (doOnNext, doOnError, doFinally)    — observe, don't alter
     ▼  Collecting       (collectList, collectMap)            — many → one collection
     ▼  Aggregation      (count, reduce, scan)                — many → one summary
     ▼  Utility          (log, checkpoint)                    — debugging aids
subscribe()
```

### One sentence to remember

> **"Don't memorize 28 operator names — recognize the family a problem belongs
> to ('I need a running total' → Aggregation), and the right operator becomes
> obvious."**
