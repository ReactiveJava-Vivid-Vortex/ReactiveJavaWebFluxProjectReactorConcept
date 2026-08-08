# When NOT to Use WebFlux

## In Simple Terms

WebFlux is powerful, but it's not automatically the right pick for every
project. Here's when you're better off sticking with traditional Spring
MVC.

## When to Avoid WebFlux

1. **Your team hasn't worked with reactive code before.** Reading reactive
   stack traces, understanding thread-switching, and writing correct
   `StepVerifier` tests all take real time to learn. If your team isn't
   ready for that, the added complexity may not be worth it.

2. **You depend on blocking libraries with no reactive equivalent.** If
   your data layer is JPA/Hibernate (blocking) and you're not planning to
   move to R2DBC anytime soon, you'd end up wrapping every call with
   `subscribeOn(boundedElastic())` — at which point you barely get any of
   WebFlux's real scalability benefit, while still paying for its extra
   complexity.

3. **Your workload is CPU-bound, not I/O-bound.** Reactive programming's
   main advantage — using threads efficiently while waiting on I/O —
   doesn't help CPU-heavy work like image processing or big calculations.

4. **Your concurrency needs are small.** An internal admin tool used by a
   handful of employees doesn't need WebFlux's scalability — the extra
   complexity just isn't worth it there.

## Simple Example

```
Decision Guide:
- High concurrency + I/O-heavy + reactive-ready dependencies (R2DBC, WebClient)
  -> WebFlux is a strong fit

- Low/moderate concurrency, OR heavy reliance on blocking libraries (JPA),
  OR mostly CPU-bound work, OR team unfamiliar with reactive
  -> Spring MVC is likely the simpler, equally effective choice
```

## Why It Matters

Picking WebFlux just "because it's more scalable," without weighing these
factors, often leads to a system that's harder to build and maintain
without actually getting the scalability payoff — because blocking
dependencies underneath undermine the whole point.
