# Q1. Why Not Just Process One Item at a Time?

## Simple Explanation (Think of Grocery Checkout)

Imagine a cashier scanning items **one at a time** and running to the back office
to log each single item in a ledger — then coming back for the next item. Absurdly
slow. A sane cashier scans a whole **basket** of items, then logs them **all at
once.**

```
One at a time:  scan -> log -> scan -> log -> scan -> log   (1000 trips!)
Batched:        scan, scan, scan, ... (100 items) -> log ALL 100 AT ONCE (1 trip!)
```

This topic is about reorganizing a stream of individual items into efficient
chunks — before they hit something expensive, like a database insert.

---

## Q2. `buffer()` — The Simplest Batching Tool

```java
Flux.range(1, 10)
    .buffer(3)
    .subscribe(batch -> System.out.println("Batch: " + batch));
```

```
Batch: [1, 2, 3]
Batch: [4, 5, 6]
Batch: [7, 8, 9]
Batch: [10]        <- last batch can be smaller!
```

```java
recordFlux
    .buffer(1000)                                  // gather up to 1000 records
    .flatMap(batch -> database.bulkInsert(batch))   // ONE bulk call instead of 1000 single ones
    .subscribe();
```

---

## Q3. What If Items Trickle In Slowly? (`bufferTimeout()`)

A size-only buffer can wait **forever** if items arrive slowly. `bufferTimeout()`
closes a batch after either the size OR a time limit — whichever comes first.

```java
Flux.interval(Duration.ofMillis(100))
    .bufferTimeout(5, Duration.ofMillis(300)) // batch of 5, OR every 300ms
    .subscribe(batch -> System.out.println("Batch: " + batch));
// Batch: [0, 1, 2]   <- only 3 items, but 300ms passed, so it closed anyway
```

---

## Q4. `buffer()` vs `window()` — What's the Real Difference?

`buffer()` gives you a fully materialized `List`. `window()` gives you a **Flux**
(a stream) for each chunk instead — useful when a chunk itself might be huge, or
you want to apply further reactive operators to it without loading it all into
memory.

```java
Flux.range(1, 9)
    .window(3)                                     // each chunk is a Flux<Integer>
    .flatMap(windowFlux -> windowFlux.reduce(0, Integer::sum)) // sum EACH window reactively
    .subscribe(sum -> System.out.println("Window sum: " + sum));
// Window sum: 6  (1+2+3)
// Window sum: 15 (4+5+6)
// Window sum: 24 (7+8+9)
```

`windowTimeout()` is to `window()` what `bufferTimeout()` is to `buffer()` — same
size-or-time cutoff logic.

---

## Q5. `groupBy()` — Splitting a Stream by Key

Like SQL's `GROUP BY` — partitions one `Flux` into multiple sub-streams
(`GroupedFlux`), one per distinct key.

```java
Flux.just("apple", "banana", "avocado", "blueberry", "cherry")
    .groupBy(word -> word.charAt(0))
    .flatMap(group -> group.collectList().map(list -> group.key() + ": " + list))
    .subscribe(System.out::println);
```

```
a: [apple, avocado]
b: [banana, blueberry]
c: [cherry]
```

**Critical gotcha:** each `GroupedFlux` MUST be subscribed to (usually via
`.flatMap()`) — an ignored group silently buffers items in memory forever, since
nothing is draining them.

---

## Q6. Interview-Style Q&A

### If a `Flux` has 10 items and I `.buffer(3)`, how many batches come out?

**4** — three batches of 3, and one final batch of just 1 (the remainder).

### Why would I ever use `window()` instead of `buffer()`?

When a "chunk" itself could be very large — `window()` lets you process it
reactively (with further operators) instead of fully materializing it as a `List`
in memory first.

### What happens if I `groupBy()` but ignore one of the resulting groups?

Its items keep buffering internally, unconsumed — a memory leak. Every
`GroupedFlux` from a `groupBy()` must be subscribed to.

---

## Q7. Summary — Which Tool for Which Job

| Need | Tool |
|---|---|
| Fixed-size chunks, as a List | `buffer(n)` |
| Fixed-size chunks, but don't wait forever on slow streams | `bufferTimeout(n, duration)` |
| Chunks as a reactive stream (not a List) | `window(n)` |
| Streamed chunks + time cutoff | `windowTimeout(n, duration)` |
| Split by KEY, not by count/time | `groupBy(keyFn)` — remember to `flatMap()` each group! |

### One sentence to remember

> **"Don't make one trip to the database per grocery item — buffer/window
> gathers a basketful first, then makes one efficient trip."**
