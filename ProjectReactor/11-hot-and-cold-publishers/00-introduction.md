# Hot & Cold Publishers — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic answers a subtle but important question: **when two subscribers
subscribe to the same `Flux`, do they each get their own independent execution, or
do they share one live, ongoing execution?**

- A **cold** publisher (the default — `Flux.just()`, a database query, an HTTP
  call) re-runs its source logic **fresh, from scratch, for every subscriber** —
  like a video-on-demand service where every viewer starts at frame zero.
- A **hot** publisher produces data **regardless of whether anyone is listening**,
  and all current subscribers share the **same** ongoing execution — like tuning
  into a live TV broadcast; you only see what airs from the moment you tune in.

```java
Flux<Long> cold = Flux.just(System.currentTimeMillis());
cold.subscribe(t -> System.out.println("A: " + t));
cold.subscribe(t -> System.out.println("B: " + t)); // DIFFERENT timestamp — re-ran!
```

You deliberately convert a cold publisher into a hot one using `.share()` (starts
when the first subscriber arrives, stops when the last leaves) or `.publish()`
(waits for an explicit `.connect()` call, giving you precise control over timing).
`.replay()` adds a twist: it remembers some history so **late** subscribers still
catch up on recent items, not just future ones.

`.refCount()` and `.autoConnect()` are the "auto-pilot" controls for a
`ConnectableFlux` (from `.publish()`) — deciding exactly when the shared source
starts and stops based on subscriber count.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Cold Publisher** | Re-runs from scratch for every subscriber — no sharing (the default for most sources). |
| 2 | **Hot Publisher** | Produces data regardless of subscribers; all current subscribers share one live execution. |
| 3 | **share()** | Converts cold → hot; starts on first subscriber, stops on last — late joiners miss earlier items. |
| 4 | **replay()** | Hot + remembers some/all history, so late subscribers catch up instead of missing everything before they joined. |
| 5 | **publish()** | Converts to a `ConnectableFlux` that won't start until you explicitly call `.connect()`. |
| 6 | **refCount()** | Auto-manages a `ConnectableFlux`'s connection: starts on first subscriber, stops on last (`share()` = `publish().refCount()`). |
| 7 | **autoConnect()** | Starts once a minimum subscriber count is reached, but — unlike `refCount()` — never auto-stops afterward. |

## How It All Fits Together

```
Should each subscriber get their OWN independent run of the source?
   │
   ├── YES (default) ────────────────────▶ leave it cold, do nothing special
   │
   └── NO, subscribers should SHARE one live execution
              │
              ├── Simple share, starts/stops automatically ──▶ .share()
              ├── Need late subscribers to catch up on history ──▶ .replay()
              └── Need precise control over WHEN it starts ──▶ .publish() + .connect()
                                                                (or .refCount() / .autoConnect()
                                                                 for automatic start/stop rules)
```

Rule of thumb: leave sources cold unless you have a specific reason (an expensive
resource, a live broadcast, a shared WebSocket) to make them hot — cold is simpler
and usually what you actually want.
