# Error Handling — Topic Overview

## What Is This Topic About? (In Simple Terms)

In a reactive pipeline, an error isn't a stack unwind like a thrown Java exception —
it's just another kind of signal (`onError`) flowing downstream, and you can
intercept and react to it at any point in the chain, just like you'd intercept a
normal value. This topic is your toolkit of `onError*` operators, each answering the
question **"what should happen when this specific step fails?"**

The three most commonly confused operators, side by side:

```java
// onErrorReturn(): swap in one static fallback VALUE, then the stream ends
mono.onErrorReturn(-1);

// onErrorResume(): switch to a whole different Mono/Flux (can itself be async)
mono.onErrorResume(e -> backupService.call());

// onErrorMap(): re-throw a DIFFERENT exception (still fails, just translated)
mono.onErrorMap(SQLException.class, e -> new ServiceException("DB failed", e));
```

For failures that might just be transient (a momentary network blip), `.retry(n)`
tries again immediately, while `.retryWhen(Retry.backoff(...))` retries with
increasing delays between attempts — always preferred for real external services, so
you don't hammer something that's already struggling. `.timeout(duration)` ensures
you never wait forever for an unresponsive dependency.

The overarching skill here is **designing layered fallbacks**: try the primary path,
retry transient failures, fall back to a cache or secondary source, and only as a
last resort return a default value — so one failing dependency degrades gracefully
instead of crashing everything.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **onErrorReturn()** | Swap in one static fallback value on error; the stream still ends after that. |
| 2 | **onErrorResume()** | Switch to a completely different (possibly async) Mono/Flux on error — most powerful/common. |
| 3 | **onErrorComplete()** | Catch the error and simply complete silently, as if nothing more was coming. |
| 4 | **onErrorMap()** | Catch an error and re-throw it as a different, more meaningful exception type. |
| 5 | **retry()** | Immediately re-subscribe up to N times on failure — no delay between attempts. |
| 6 | **retryWhen()** | Full control over retry timing/conditions, typically with exponential backoff via `Retry.backoff()`. |
| 7 | **timeout()** | Fail with `TimeoutException` if no signal arrives within a given duration — prevents indefinite hangs. |
| 8 | **fallback** | The general umbrella pattern: layering `onErrorReturn`/`onErrorResume`/`defaultIfEmpty` for resilience. |
| 9 | **Recovering from failures** | Combining timeout + retry + fallback + default into one deliberate, layered recovery strategy. |

## How It All Fits Together

```
Operation fails
      │
      ▼
Is it likely transient (network blip)?
   │
   ├── YES ──▶ retryWhen(Retry.backoff(...))  ──▶ still fails after retries?
   │                                                       │
   └── NO ─────────────────────────────────────────────────┤
                                                            ▼
                                          onErrorResume(fallback source)
                                                            │
                                          still nothing? → onErrorReturn(default value)
```

Every operator here answers "what next?" after a failure — string a few of them
together (timeout → retry → fallback → default) and you've built genuine
resilience, not just error suppression.
