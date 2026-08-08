# Q1. How Do I Combine Multiple Reactive Sources?

## Simple Explanation (Think of Two Chefs vs One Chef)

If you need results from two data sources, you have two choices:

```
One chef, one dish at a time (concat):
  Chef finishes dish A completely -> THEN starts dish B
  Slower overall, but strict order guaranteed

Two chefs, working at the same time (merge/zip):
  Chef 1 makes dish A, Chef 2 makes dish B, SIMULTANEOUSLY
  Faster overall — total time ≈ the SLOWER of the two, not the sum
```

Picking the right combination operator is really just answering: **"do these need
to happen in order, or can they happen at the same time?"**

---

## Q2. `concat()` vs `merge()` — The Core Distinction

```java
Flux<Integer> first = Flux.just(1, 2, 3);
Flux<Integer> second = Flux.just(4, 5, 6);

Flux.concat(first, second).subscribe(n -> System.out.println("concat: " + n));
// 1, 2, 3, 4, 5, 6 — STRICT order, second doesn't even START until first completes

Flux.merge(first, second).subscribe(n -> System.out.println("merge: " + n));
// interleaved based on actual timing — order NOT guaranteed
```

| | `concat()` | `merge()` |
|---|---|---|
| Subscription | Sequential (one at a time) | Concurrent (all sources at once) |
| Order | Strictly preserved | Interleaved by timing |
| Speed | Slower (waits for each) | Faster (parallel work) |

---

## Q3. What If I Want BOTH Speed AND Order? (`mergeSequential()`)

```java
Flux<String> fast = Flux.just("A1", "A2").delayElements(Duration.ofMillis(50));
Flux<String> slow = Flux.just("B1", "B2").delayElements(Duration.ofMillis(200));

Flux.mergeSequential(fast, slow).subscribe(item -> System.out.println(item));
// A1, A2, B1, B2 — GUARANTEED order, even though BOTH started working immediately
```

Both sources start working **concurrently** the instant you subscribe — but
`fast`'s results are buffered internally until it's actually `fast`'s turn to
emit, preserving source order.

---

## Q4. How Do I Pair Up Results from Parallel Calls? (`zip()`)

```java
Mono<UserProfile>   profile = userService.getProfile(userId);
Mono<List<Order>>   orders  = orderService.getOrders(userId);

Mono.zip(profile, orders)     // BOTH calls happen IN PARALLEL
    .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()))
    .subscribe(dashboard -> render(dashboard));
```

`zip()` waits until **all** sources have produced their Nth item, then pairs them
together — and stops as soon as the **shortest** source runs out.

```java
Flux<String> names = Flux.just("Alice", "Bob");        // only 2 items
Flux<Integer> ages  = Flux.just(30, 25, 35);             // 3 items

Flux.zip(names, ages).subscribe(t -> System.out.println(t));
// Only 2 pairs — "Charlie"/35 dropped, since names ran out first!
```

---

## Q5. `zip()` vs `combineLatest()` — Another Classic Mix-Up

```java
Flux<String> temp = Flux.just("20C", "22C", "25C").delayElements(Duration.ofMillis(100));
Flux<String> hum  = Flux.just("40%", "45%").delayElements(Duration.ofMillis(150));

Flux.combineLatest(temp, hum, (t, h) -> t + " / " + h)
    .subscribe(System.out::println);
// Recombines using the LATEST value from each source, whenever EITHER updates —
// NOT a strict 1-to-1 pairing like zip()
```

| | `zip()` | `combineLatest()` |
|---|---|---|
| Pairing | Strict 1-to-1, in order | Latest-known value from each source |
| Emits when | ALL sources have a new item | ANY source emits a new item |
| Best for | Combining independent parallel results | Live dashboards, continuously-updating sources |

---

## Q6. What If One Source Might Fail, but I Don't Want to Lose the Others?

```java
Flux<Integer> first = Flux.just(1, 2).concatWith(Flux.error(new RuntimeException("boom")));
Flux<Integer> second = Flux.just(3, 4);

Flux.concatDelayError(first, second)
    .subscribe(
        n -> System.out.println("Got: " + n),
        e -> System.out.println("Error at the end: " + e.getMessage())
    );
// Got: 1, Got: 2, Got: 3, Got: 4, THEN the error — plain concat() would have
// stopped immediately after "2", never even trying `second`
```

`mergeDelayError()` does the same thing for concurrent sources — every source gets
a chance to finish before any error is surfaced.

---

## Q7. How Do I "Race" Multiple Sources and Take the Fastest?

```java
Mono<String> serverA = callServer("A").delayElement(Duration.ofMillis(200));
Mono<String> serverB = callServer("B").delayElement(Duration.ofMillis(100));

Mono.firstWithSignal(serverA, serverB)
    .subscribe(result -> System.out.println("Winner: " + result));
// "Winner: Response from B" — A's call is cancelled once B wins
```

Use `firstWithValue()` instead if a source might legitimately complete **empty**
(like a cache miss) — `firstWithSignal()` would wrongly let an empty result "win"
the race.

---

## Q8. Interview-Style Q&A

### If I call `Flux.concat(a, b)`, does `b` start before `a` finishes?

**No.** `concat()` guarantees `a` fully completes before `b` is even subscribed
to.

### Does `zip()` wait for the slowest or fastest source?

Effectively the **slowest**, since it must wait for all sources to produce their
Nth item before emitting a pair — but it stops entirely as soon as the
**shortest** source runs out of items.

### What's the practical difference between `merge()` and `mergeSequential()`?

Both subscribe concurrently, but `mergeSequential()` reorders the output to match
source order; `merge()` emits in whatever order items actually arrive.

---

## Q9. Summary — The Decision Table

| Need | Operator |
|---|---|
| Strict order, one at a time | `concat()` / `concatWith()` |
| Strict order, but don't fail early on error | `concatDelayError()` |
| Fastest, order doesn't matter | `merge()` |
| Fastest, but need result order preserved | `mergeSequential()` |
| Pair up parallel results 1-to-1 | `zip()` / `zipWith()` |
| React to "latest value from each" continuously | `combineLatest()` |
| Race sources, take whichever responds first | `firstWithSignal()` / `firstWithValue()` |

### One sentence to remember

> **"concat() = one chef at a time; merge()/zip() = two chefs working
> simultaneously — pick based on whether order matters more than speed."**
