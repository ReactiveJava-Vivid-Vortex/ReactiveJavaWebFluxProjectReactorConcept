# Q1. What Is Project Reactor?

## Simple Explanation (Think of a Recipe vs Actually Cooking)

A `Mono`/`Flux` pipeline is like a **recipe card**. Writing the recipe
(`Flux.just(...).map(...).filter(...)`) doesn't cook anything — it just describes
steps. Food only actually gets made when someone decides to **cook it**
(`.subscribe()`).

```
Write the recipe   -> Flux.just(1,2,3).map(n -> n * 2)     (nothing happens)
Start cooking       -> .subscribe(...)                       (NOW it runs)
```

**Project Reactor** is the library that gives you the "recipe cards" — `Mono` (0 or
1 result) and `Flux` (0 to N results) — plus hundreds of operators, all
implementing the Reactive Streams contract correctly and safely so you don't have
to hand-write it yourself.

---

## Q2. What's the Setup?

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.0</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>  <!-- for StepVerifier -->
    <version>3.6.0</version>
    <scope>test</scope>
</dependency>
```

If you're on Spring Boot WebFlux, this comes free — `spring-boot-starter-webflux`
pulls in `reactor-core` transitively.

---

## Q3. Why Doesn't My Code Run? (Lazy Execution)

This is **the** most common "wait, what?" moment for beginners.

```java
System.out.println("1. Before building pipeline");

Mono<String> mono = Mono.fromSupplier(() -> {
    System.out.println("3. Supplier is running!"); // does NOT print yet
    return "Hello";
});

System.out.println("2. Pipeline built — nothing ran above yet");

mono.subscribe(value -> System.out.println("4. Got: " + value));
```

Output:
```
1. Before building pipeline
2. Pipeline built — nothing ran above yet
3. Supplier is running!
4. Got: Hello
```

Notice line "3" prints **after** line "2" — building a `Mono`/`Flux` is just
describing what *should* happen; nothing executes until `.subscribe()`.

---

## Q4. What's a "Cold" Publisher, and Why Should I Care?

A **cold** publisher re-runs its source logic **fresh, for every subscriber** —
like a video-on-demand service where every viewer starts at frame zero. Most
sources you create (`Flux.just()`, a database query) are cold by default.

```java
Mono<Long> coldMono = Mono.just(System.currentTimeMillis());

coldMono.subscribe(t -> System.out.println("Subscriber 1: " + t));
Thread.sleep(2000);
coldMono.subscribe(t -> System.out.println("Subscriber 2: " + t));
```

```
Subscriber 1: 1732000000000
Subscriber 2: 1732000002000   <- DIFFERENT timestamp — re-ran the source!
```

(This is only half the story — the full "Hot vs Cold" comparison, with live-TV
analogies, sharing, and replay, gets its own dedicated deep-dive topic later in
this course.)

---

## Q5. What Is a "Reactive Pipeline," Really?

A chain of operators — an assembly line data flows through, **once subscribed**.

```java
Flux<String> pipeline = Flux.just("apple", "banana", "cherry")
    .filter(f -> f.length() > 5)
    .map(String::toUpperCase)
    .doOnNext(f -> System.out.println("About to emit: " + f));

// NOTHING has happened yet — it's still just a description

pipeline.subscribe(f -> System.out.println("Received: " + f));
```

```
About to emit: BANANA
Received: BANANA
About to emit: CHERRY
Received: CHERRY
```

Notice each item flows through the **entire** pipeline one at a time (filter → map
→ doOnNext → subscribe), rather than "filter everything, then map everything."

---

## Q6. What Is the Subscription Model? (Demand Flows Up, Data Flows Down)

```
.subscribe() called
        │
        ▼
request(n) travels UPSTREAM   (subscriber -> ... -> source)
        │
        ▼
onNext() data travels DOWNSTREAM   (source -> ... -> subscriber)
```

```java
Flux.range(1, 3)
    .doOnSubscribe(s -> System.out.println("1. Subscribed"))
    .doOnRequest(n -> System.out.println("2. Requested: " + n))
    .doOnNext(v -> System.out.println("3. Emitting: " + v))
    .subscribe(v -> System.out.println("4. Received: " + v));
```

```
1. Subscribed
2. Requested: 9223372036854775807
3. Emitting: 1
4. Received: 1
3. Emitting: 2
4. Received: 2
...
```

Subscription and demand always happen **first** (traveling up), and only then does
data flow back down — everything starts from `.subscribe()`.

---

## Q7. How Do I See What's Actually Happening? (`.log()`)

```java
Flux.just(1, 2, 3)
    .log()
    .map(n -> n * 2)
    .subscribe();
```

```
[ INFO] onSubscribe(FluxArray.ArraySubscription)
[ INFO] request(unbounded)
[ INFO] onNext(1)
[ INFO] onNext(2)
[ INFO] onNext(3)
[ INFO] onComplete()
2
4
6
```

`.log()` is the fastest way to answer "did it even subscribe? how much was
requested? did the error happen before or after this operator?"

---

## Q8. Interview-Style Q&A

### If I never call `.subscribe()`, does anything happen?

**No.** Nothing at all — not even side effects inside `.map()`/`.doOnNext()`
lambdas. This is the #1 cause of "my reactive code silently does nothing" bugs.

### If I subscribe to the same cold `Mono` twice, do I get the same result?

**Not necessarily.** Each subscription re-runs the source from scratch — if the
source is time-sensitive (like `System.currentTimeMillis()`) or has side effects
(like a DB write), each subscriber can see different results.

### Does `Mono.just(value)` re-compute `value` per subscriber?

**No** — `Mono.just()` captures its value **eagerly** at creation time (a common
trap: `Mono.just(expensiveCall())` runs `expensiveCall()` immediately, even with no
subscriber). Use `Mono.fromSupplier()` for lazy, per-subscription evaluation.

---

## Q9. Summary

| Concept | Key Takeaway |
|---|---|
| Project Reactor | The library implementing Reactive Streams via `Mono`/`Flux` + operators |
| Setup | `reactor-core` (+ `reactor-test` for `StepVerifier`) — free with Spring WebFlux |
| Logging | `.log()` prints every signal — the best debugging tool you have |
| Reactive pipeline | A blueprint of operators; inert until subscribed |
| Cold publishers | Re-run fresh per subscriber, by default (full Hot/Cold topic later) |
| Lazy execution | Nothing runs until `.subscribe()` — building ≠ running |
| Subscription model | Demand flows UP first, data flows DOWN in response |

### One sentence to remember

> **"A Mono/Flux is a recipe, not a meal — nothing gets cooked until someone
> subscribes."**
