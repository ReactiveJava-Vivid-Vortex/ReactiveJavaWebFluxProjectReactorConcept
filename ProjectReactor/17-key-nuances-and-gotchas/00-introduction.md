# Key Nuances & Gotchas — Topic Overview

## What Is This Topic About? (In Simple Terms)

This bonus topic isn't from the original course outline — it's a collection of the
small, foundational realizations that make everything else in this course click
faster. Each one is a common trap or a "wait, really?" moment that almost every
newcomer to reactive programming hits, explained once and for all so you recognize
it instantly next time.

If you remember only one idea from this whole folder, make it this: **a
`Mono`/`Flux` is an immutable, lazy *description* of work, not the work itself.**
Everything else here — assembly vs. subscription time, why an unassigned `.map()`
call does nothing, why errors stop the whole stream — is really just that one idea,
viewed from a different angle.

```java
Flux<Integer> numbers = Flux.just(1, 2, 3);

numbers.map(n -> n * 100); // BUG: return value thrown away, does NOTHING

Flux<Integer> scaled = numbers.map(n -> n * 100); // FIX: capture the new Flux
```

The other big idea in this folder is the **three-signal grammar**
(`onSubscribe onNext* (onError | onComplete)?`) introduced in the Reactive Streams
Specification topic — several nuances here (Mono-as-capped-Flux, why errors
terminate everything, why signals never race) are direct, practical consequences of
that one grammar rule.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Operators return new instances** | `Mono`/`Flux` are immutable — an unassigned/unchained `.map()` call is a silent no-op. |
| 2 | **Assembly time vs subscription time** | Building a pipeline ≠ running it; lambda bodies inside operators only run once subscribed. |
| 3 | **Signals are sequential, never concurrent** | Reactive Streams guarantees one signal at a time per subscription — no extra locking needed inside a subscriber. |
| 4 | **Mono is a Flux of at-most-one** | Same rules, same operators — Mono just caps `onNext` at 0-or-1 instead of 0-to-N. |
| 5 | **subscribe() method overloads** | Always supply an error consumer — otherwise failures are silently logged by Reactor, not handled by your code. |
| 6 | **Errors terminate the entire chain** | One bad item can abort an entire Flux — isolate per-item errors with `flatMap` + `onErrorResume` if you need to skip-and-continue. |

## How It All Fits Together

```
Mono/Flux = an immutable, lazy DESCRIPTION of work
        │
        ├── Assembly time: pipeline is built, described, NOT executed
        │
        └── Subscription time: .subscribe() triggers actual execution
                    │
                    ▼
        Signals flow, one at a time, per subscription:
        onSubscribe → onNext* → (onError | onComplete)?
                    │
        ├── onNext repeats (0 to N for Flux, 0-or-1 for Mono)
        └── terminal signal ENDS EVERYTHING — no partial recovery
            without explicit per-item error handling
```

Keep this folder as your "when something feels weird, check here first" reference
— most reactive confusion traces back to one of these six ideas.
