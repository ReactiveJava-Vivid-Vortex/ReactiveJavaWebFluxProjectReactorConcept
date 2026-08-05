# When NOT to Use WebFlux

## In Simple Terms

WebFlux is a powerful tool, but it's not automatically the right choice for every
project. Here's when to prefer traditional Spring MVC instead.

## When to Avoid WebFlux

1. **Your team lacks reactive programming experience.** Debugging reactive stack
   traces, understanding thread-switching behavior, and writing correct
   `StepVerifier` tests all have a real learning curve. If your team isn't ready for
   that investment, the complexity cost may outweigh the benefit.

2. **You rely on blocking libraries with no reactive equivalent.** If your data
   access layer is JPA/Hibernate (blocking) and there's no near-term plan to migrate
   to R2DBC, you'd end up wrapping every call with `subscribeOn(boundedElastic())`
   — at that point, you gain little of WebFlux's actual scalability benefit while
   still paying its complexity cost.

3. **Your workload is CPU-bound, not I/O-bound.** Reactive programming's main
   advantage (efficient thread usage during I/O waits) doesn't help CPU-heavy
   workloads like image processing or complex calculations.

4. **Your concurrency needs are low.** An internal admin tool used by a handful of
   employees doesn't need WebFlux's scalability — the added complexity isn't worth
   it.

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

Choosing WebFlux "because it's more scalable" without considering these factors
often leads to a system that's more complex to build and maintain, without actually
realizing the scalability benefits — because the underlying blocking dependencies
undermine the whole point.
