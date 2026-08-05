# Key Nuances & Gotchas — Topic Overview

## What Is This Topic About? (In Simple Terms)

This bonus topic collects the WebFlux-specific "wait, really?" moments — the traps
and misconceptions that catch people who otherwise understand Project Reactor well,
but haven't yet hit these specific Spring integration quirks.

The single most dangerous one: **`.block()` exists and compiles fine, but calling
it inside a controller/filter can throw at runtime — and even when it doesn't
throw, it silently defeats the entire point of using WebFlux** by freezing one of
your few event-loop threads.

```java
// Throws IllegalStateException in many cases, and even when it doesn't,
// it undermines everything WebFlux is for:
return productRepository.findById(id).block(); // DON'T
```

A close second: assuming empty results and error handling behave "the way you'd
expect" by default. An empty `Mono` returned from a controller does **not**
automatically become a `404` — you have to say so explicitly. And `@Valid`
doesn't reliably fire on a `Mono<T>`-wrapped request body the way it does on a
plain `T` — another "looks like it should just work" trap.

The reassuring flip side: WebFlux is more flexible than the purist rules suggest —
you *can* mix functional and annotated endpoints in the same app, and you *can*
safely use an unavoidable blocking library, as long as you isolate it correctly on
`boundedElastic()`.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Empty Mono ≠ automatic 404** | Returning an empty Mono from a controller defaults to `200 OK` with an empty body — you must opt into 404 explicitly. |
| 2 | **Never call .block() on WebFlux threads** | Freezes an event-loop thread (may even throw `IllegalStateException`) — always return the Mono/Flux instead. |
| 3 | **@RequestBody T vs Mono&lt;T&gt;** | Plain `T` reliably triggers `@Valid`; `Mono<T>` does not — validate explicitly if you use the Mono form. |
| 4 | **Functional & annotated endpoints coexist** | You can freely mix `@RestController` and `RouterFunction` styles in the same application. |
| 5 | **Isolating unavoidable blocking calls** | A few well-isolated (`boundedElastic()`) blocking calls don't disqualify an app from being "properly reactive." |

## How It All Fits Together

```
Writing a WebFlux endpoint
        │
        ├── Handling "not found"?  ──▶ explicitly convert empty → 404 (don't assume it's automatic)
        │
        ├── Need the actual value right now?  ──▶ NEVER .block() — return Mono/Flux instead
        │
        ├── Validating a request body?  ──▶ @Valid works on plain T; verify it separately for Mono<T>
        │
        └── Have an unavoidable blocking dependency?
                    │
                    ├── isolate it: .subscribeOn(Schedulers.boundedElastic())
                    └── the rest of the app can still be genuinely reactive
```

Keep this list handy for code review — most of these are exactly the kind of subtle
bug that passes a quick manual test but causes confusing behavior (or a production
incident) under real traffic.
