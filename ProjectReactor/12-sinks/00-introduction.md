# Sinks — Topic Overview

## What Is This Topic About? (In Simple Terms)

Sometimes you need to manually **push** values into a reactive stream from
arbitrary code — not from a database query or an HTTP call, but from, say, an
event handler or a background thread. `Sinks` is Reactor's modern, safe API for
exactly this: it's a "microphone" you hold onto, and you can shout values into it
from anywhere, while a `Flux`/`Mono` on the other end lets subscribers listen.

`Sinks` replaced the older `Processor` API because it's much harder to misuse: every
emission method (`tryEmitNext()`, `tryEmitComplete()`, `tryEmitError()`) returns an
explicit `EmitResult` you can check, instead of throwing unpredictable exceptions.

```java
Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

sink.asFlux().subscribe(msg -> System.out.println("Got: " + msg));

sink.tryEmitNext("Hello from anywhere in the code!");
```

The most important decision when creating a `Sinks.Many` is the **distribution
strategy**:

- **Multicast** — broadcasts to all *currently* subscribed listeners (like a live
  radio broadcast; late joiners miss earlier messages).
- **Unicast** — supports exactly one subscriber, buffering until it arrives.
- **Replay** — like multicast, but remembers some history for late-joining
  subscribers too.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Sinks.One** | Modern way to manually produce a single value — the programmatic version of a `Mono`. |
| 2 | **Sinks.Many** | Modern way to manually produce a stream of values — the programmatic version of a `Flux`. |
| 3 | **Multicast** | Broadcasts to all *current* subscribers at once — late joiners miss earlier emissions. |
| 4 | **Unicast** | Supports only ONE subscriber at a time, buffering emissions until it shows up. |
| 5 | **Replay** | Multicast + remembers some/all history so late subscribers can catch up. |
| 6 | **Direct best effort** | No buffering at all — slow subscribers simply miss emissions (best-effort delivery). |
| 7 | **Event broadcasting** | The overall pattern: use a `Sinks.Many` as a lightweight, in-process pub/sub event bus. |
| 8 | **Producer APIs** | `tryEmitNext/Complete/Error()` return an `EmitResult` you should check — never silently ignore failures. |

## How It All Fits Together

```
How many subscribers, and do late joiners need history?
   │
   ├── Exactly ONE subscriber ─────────────────▶ Sinks.many().unicast()
   │
   ├── MANY subscribers, only care about NOW ──▶ Sinks.many().multicast()
   │
   └── MANY subscribers, late joiners need
       recent history too ────────────────────▶ Sinks.many().replay()

Producing:  sink.tryEmitNext(value)  →  check the returned EmitResult!
Consuming:  sink.asFlux().subscribe(...)   (or .asMono() for Sinks.One)
```

`Sinks` is the bridge between "the reactive world" and "arbitrary imperative code
that needs to push events in" — you'll see this pattern again in Spring WebFlux when
building Server-Sent Events endpoints.
