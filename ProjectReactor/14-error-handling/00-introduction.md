# Q1. How Is Reactive Error Handling Different from try/catch?

## Simple Explanation (Think of a Relay Baton, Not a Landmine)

In imperative code, an exception is like a landmine — it explodes and unwinds the
stack immediately. In reactive code, an error is just another kind of **signal**
(`onError`) passed down the pipeline like a baton — and you can intercept, inspect,
and react to that baton at any handoff point, the same way you'd react to a normal
value.

```java
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)   // throws when n == 0
    .subscribe(
        v -> System.out.println("Value: " + v),
        e -> System.out.println("Caught error: " + e.getMessage())
    );
// Value: 10, Value: 5, Caught error: / by zero  (4 is NEVER processed)
```

---

## Q2. The Three Most Confused Operators, Side by Side

```java
// onErrorReturn(): swap in ONE static fallback VALUE — stream still ends after
mono.onErrorReturn(-1);

// onErrorResume(): switch to a WHOLE different Mono/Flux (can be async)
mono.onErrorResume(e -> backupService.call());

// onErrorMap(): re-throw a DIFFERENT exception — still fails, just translated
mono.onErrorMap(SQLException.class, e -> new ServiceException("DB failed", e));
```

| Operator | What Happens | Stream Still Ends in Error? |
|---|---|---|
| `onErrorReturn()` | Emits one fallback value | No — becomes a success |
| `onErrorResume()` | Switches to a different publisher | Depends on the fallback |
| `onErrorMap()` | Translates the exception type | Yes — just a different exception |
| `onErrorComplete()` | Silently completes, no value | No — becomes empty success |

---

## Q3. How Do I Retry a Failed Operation?

```java
// retry(n): immediate re-subscribe, NO delay between attempts
mono.retry(2);

// retryWhen(): full control — exponential backoff, filtering which errors to retry
mono.retryWhen(
    Retry.backoff(3, Duration.ofMillis(100))
        .maxBackoff(Duration.ofSeconds(2))
        .filter(e -> e instanceof TimeoutException) // only retry timeouts
);
```

**Prefer `retryWhen()` with backoff for real external services** — plain
`retry()`'s zero-delay retries can hammer an already-struggling downstream
dependency.

---

## Q4. How Do I Stop Waiting Forever?

```java
mono.timeout(Duration.ofSeconds(2))
    .onErrorResume(TimeoutException.class, e -> Mono.just("Fallback: took too long"))
    .subscribe(System.out::println);
```

Without `.timeout()`, a single unresponsive dependency can hang a request
indefinitely, eventually exhausting resources across the whole application.

---

## Q5. How Do I Design a Real, Layered Recovery Strategy?

```java
public Mono<Price> getPrice(String productId) {
    return primaryPriceService.getPrice(productId)
        .timeout(Duration.ofSeconds(1))
        .onErrorResume(error -> cachedPriceService.getPrice(productId)) // fallback #1: cache
        .switchIfEmpty(Mono.just(Price.DEFAULT))                        // fallback #2: default
        .onErrorReturn(Price.UNAVAILABLE);                               // last resort
}
```

Try the live service (bounded) → fall back to cache on failure → fall back to a
sensible default if there's genuinely no data → absolute last resort, a static
marker instead of propagating any error to the caller.

---

## Q6. Interview-Style Q&A

### Does `onErrorResume()` let the original stream "continue" after the error?

**No.** The original stream is still over — `onErrorResume()` switches to a
**brand new** publisher, which starts its own fresh signal sequence.

### If I don't provide an error handler in `.subscribe()`, what happens?

Reactor logs the error internally, but your own application logic never gets a
chance to react to it — always supply an error consumer for anything beyond
trivial/test code.

### Why prefer `retryWhen(Retry.backoff(...))` over plain `retry(n)`?

Plain `retry()` retries instantly with no delay, which can worsen load on an
already-struggling service. Backoff spaces out retries, giving the dependency room
to recover.

---

## Q7. Summary

```
Operation fails
      │
      ▼
Is it likely transient (network blip)?
   │
   ├── YES ──▶ retryWhen(Retry.backoff(...)) ──▶ still fails after retries?
   │                                                       │
   └── NO ─────────────────────────────────────────────────┤
                                                            ▼
                                          onErrorResume(fallback source)
                                                            │
                                          still nothing? → onErrorReturn(default value)
```

### One sentence to remember

> **"An error is a baton passed downstream, not a landmine — string together
> timeout → retry → fallback → default, and you've built genuine resilience,
> not just error suppression."**
