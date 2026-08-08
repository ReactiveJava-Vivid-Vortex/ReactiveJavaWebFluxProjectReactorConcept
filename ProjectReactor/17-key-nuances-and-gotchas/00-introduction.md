# Q1. What Is This Bonus Topic For?

## Simple Explanation (Think of the "Gotchas" Page in a Language Tutorial)

Every course has that one page titled "common mistakes" that somehow explains more
than the previous ten chapters combined. This topic is that page for Project
Reactor — six specific "wait, really?" moments that catch almost everyone once,
collected so you only have to be surprised by each of them a single time.

```
Mono/Flux = an immutable, LAZY DESCRIPTION of work, not the work itself
```

If you remember only that one sentence, most of the nuances below fall out of it
naturally.

---

## Q2. "I called `.map()` but nothing changed!" — Why?

`Mono`/`Flux` are **immutable**. `.map()` doesn't modify the original — it returns
a **new** instance. If you don't capture or chain that return value, your
transformation is a silent no-op.

```java
Flux<Integer> numbers = Flux.just(1, 2, 3);

numbers.map(n -> n * 100);   // BUG: return value thrown away!
numbers.subscribe(System.out::println); // 1, 2, 3 — the *100 never happened

Flux<Integer> scaled = numbers.map(n -> n * 100); // FIX: capture it
scaled.subscribe(System.out::println); // 100, 200, 300
```

---

## Q3. "My print statement ran at the wrong time!" — Assembly vs Subscription

```java
System.out.println("1. Assembly starts");

Flux<String> pipeline = Flux.just("a", "b")
    .map(s -> {
        System.out.println("3. Runs at SUBSCRIPTION time: " + s);
        return s.toUpperCase();
    });

System.out.println("2. Assembly finished — nothing executed yet");
pipeline.subscribe(s -> System.out.println("4. Received: " + s));
```

Code **outside** an operator lambda runs once, immediately, at assembly time. Code
**inside** an operator lambda runs later, per item, at subscription time — maybe
never, maybe many times.

---

## Q4. "Do I Need to Synchronize Inside My Subscriber?" — No

Reactive Streams guarantees signals to a single subscription are delivered
**sequentially, never concurrently** (spec Rule 1.3) — even if upstream work
happens across multiple threads.

```java
Flux.range(1, 1000)
    .parallel(4).runOn(Schedulers.parallel())  // work happens on 4 threads...
    .map(n -> n * n)
    .sequential()                               // ...but re-joined into ONE sequential stream
    .subscribe(n -> {
        // called ONE AT A TIME — no synchronization needed here
    });
```

---

## Q5. "Is Mono a Totally Different Type from Flux?" — Not Really

A `Mono<T>` is just a `Flux<T>` with `onNext` capped at *at most once*. Same
operators, same rules, same laziness — just a different cardinality ceiling.

```java
Flux<String> asFlux = mono.flux();        // Mono -> Flux
Mono<String> firstOnly = flux.next();      // Flux -> Mono (first item only)
Mono<List<String>> all = flux.collectList(); // Flux -> Mono (ALL items, as one list)
```

---

## Q6. "Why Didn't My Error Handler Fire?" — subscribe() Overloads

```java
mono.subscribe(value -> handle(value));
// If this Mono errors, YOUR code never sees it — Reactor just logs it internally!

mono.subscribe(
    value -> handle(value),
    error -> handleError(error) // NOW you're actually handling it
);
```

Always supply an error consumer in production code, unless you're certain the
source can't fail.

---

## Q7. "One Bad Item Killed My Whole Batch!" — Errors Terminate Everything

```java
Flux.just(1, 2, 0, 4, 5)
    .map(n -> 10 / n) // throws on n == 0
    .subscribe(v -> System.out.println("Got: " + v), e -> System.out.println("Ended: " + e));
// Got: 10, Got: 5, Ended: / by zero — 4 and 5 are NEVER attempted!
```

If you want "skip the bad item, keep going," isolate errors **per item**, usually
inside `flatMap`:

```java
Flux.just(1, 2, 0, 4, 5)
    .flatMap(n -> Mono.fromCallable(() -> 10 / n)
        .onErrorResume(e -> Mono.empty())) // only THIS item is skipped
    .subscribe(v -> System.out.println("Got: " + v));
```

---

## Q8. Interview-Style Q&A

### Why is `flux.filter(x -> x > 5);` on its own line always a bug?

Because the returned, filtered `Flux` is discarded — `Mono`/`Flux` are immutable,
so operators never mutate the original in place.

### Can two `onNext()` calls for the SAME subscription ever race each other?

**No** — Reactive Streams guarantees sequential delivery per subscription. Two
*different* subscriptions can still run concurrently with each other, though.

### If an item throws inside `.map()` partway through a `Flux` of 100 items, how many get processed?

Only the ones **before** the failure — the whole stream terminates at that point
unless you've isolated error handling per-item (e.g., via `flatMap` +
`onErrorResume`).

---

## Q9. Summary

| # | Nuance | One-Line Fix |
|---|---|---|
| 1 | Operators return new instances | Always capture/chain the result |
| 2 | Assembly time vs subscription time | Lambda bodies run later, not when written |
| 3 | Signals are sequential, never concurrent | No extra locking needed inside one subscriber |
| 4 | Mono is a Flux of at-most-one | Same operators, same rules, different cap |
| 5 | subscribe() overloads | Always pass an error consumer |
| 6 | Errors terminate the entire chain | Isolate per-item errors via flatMap + onErrorResume |

### One sentence to remember

> **"When something in reactive code feels weird, ask: is this an immutability
> trap, a timing trap, or a 'the whole stream just died' trap? — it's almost
> always one of the three."**
