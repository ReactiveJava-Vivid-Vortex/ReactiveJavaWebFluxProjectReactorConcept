# Project Reactor Fundamentals — Topic Overview

## What Is This Topic About? (In Simple Terms)

Now that you understand the Reactive Streams *rulebook* (Publisher/Subscriber/
Subscription), it's time to meet the library that actually implements it for you in
production-quality Java: **Project Reactor**. Instead of hand-writing
`Publisher`/`Subscriber` code (which is tedious and easy to get wrong), Reactor gives
you two ready-made types — `Mono` (0 or 1 item) and `Flux` (0 to N items) — plus
hundreds of operators to transform and combine them.

The single most important habit to build here is understanding **laziness**:
building a `Mono`/`Flux` pipeline (with `.map()`, `.filter()`, etc.) doesn't run
anything — it just describes *what should happen*. Nothing actually executes until
someone calls `.subscribe()`. This trips up almost every beginner: "why didn't my
code run?" — because nobody subscribed yet.

```java
Mono<String> mono = Mono.fromSupplier(() -> {
    System.out.println("This won't print yet!");
    return "Hello";
});

System.out.println("Pipeline built — nothing happened above this line yet.");

mono.subscribe(value -> System.out.println("Got: " + value));
// ONLY NOW does "This won't print yet!" actually print.
```

Related to laziness is the idea of **cold publishers**: most sources (like a
database query wrapped in a `Mono`) re-run their logic **fresh for every new
subscriber** — like a video-on-demand service where every viewer starts from frame
zero, rather than a live broadcast everyone shares.

Finally, understanding the **subscription model** — that demand (`request(n)`)
travels *upstream* first, and only then does data flow *downstream* — explains why
everything in Reactor is driven from the bottom (`.subscribe()`) up.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Project Reactor** | The library implementing Reactive Streams via `Mono`/`Flux` + hundreds of operators — the engine under Spring WebFlux. |
| 2 | **Maven/Gradle setup** | Add `reactor-core` (+ `reactor-test` for `StepVerifier`); comes free with `spring-boot-starter-webflux`. |
| 3 | **Logging** | `.log()` prints every signal (`onSubscribe`, `request`, `onNext`, `onComplete`/`onError`) flowing through a pipeline point. |
| 4 | **Reactive pipeline** | A chain of operators describing what should happen to data — a blueprint that only runs once subscribed. |
| 5 | **Cold publishers** | Re-run their source logic fresh for every new subscriber (like video-on-demand) — the default behavior. |
| 6 | **Lazy execution** | Nothing in a `Mono`/`Flux` chain runs until `.subscribe()` is called — building it is not running it. |
| 7 | **Subscription model** | Demand (`request(n)`) flows upstream first; data (`onNext`) flows downstream in response — everything starts from `.subscribe()`. |

## How It All Fits Together

```
You build a pipeline:      Flux.just(...).map(...).filter(...)
                                    │  (nothing runs yet — it's lazy)
                                    ▼
You call .subscribe()  ──▶  Subscription created
                                    │
                     demand flows UPSTREAM (request(n))
                                    │
                     data flows DOWNSTREAM (onNext × n)
                                    │
                                    ▼
                        Terminal signal: onComplete / onError
```

Every subtopic here is really explaining the same core truth: **a Reactor pipeline
is inert until subscribed, and each subscription can trigger its own fresh
execution.** Once that's second nature, `Mono` and `Flux` (the next two topics) will
make much more sense.
