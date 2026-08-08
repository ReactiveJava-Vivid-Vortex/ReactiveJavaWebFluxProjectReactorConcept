# When to Use Reactive Programming

## In Simple Terms

Reactive programming is a **great choice** when your app:

- Handles a **lot of requests at the same time**.
- Spends most of its time **waiting** — on databases, other services, message
  queues — rather than crunching numbers.
- Needs to stay steady even when traffic suddenly spikes.
- Deals with **streaming** data (big files, live feeds) instead of loading
  everything into memory up front.

It's a **poor choice** when your app:

- Is mostly doing **heavy computation** (image processing, number crunching) —
  reactive doesn't make CPU work go any faster.
- Only has a **handful of users** (a small internal tool used by 5 people).
- Needs to stay simple, and the team doesn't have reactive experience yet — the
  learning curve is real.

## Simple Example

```
Use Case                                   Good Fit for Reactive?
------------------------------------------ -----------------------
Public API gateway, 50k req/sec            Yes — lots of waiting, lots of traffic
Streaming stock prices to browsers (SSE)   Yes — built for streaming
Batch job resizing 10,000 images (CPU)     No — CPU-bound, reactive adds nothing
Small internal CRUD tool, 10 users         No — not worth the extra complexity
Microservice aggregating 5 downstream APIs Yes — parallel non-blocking calls shine
```

## Why It Matters

Reactive programming isn't "always the better choice" — it's a trade-off. You get
better scaling and steadier behavior under heavy I/O load, but you pay for it
with a steeper learning curve, trickier debugging, and more work to test
properly. Use it where its strengths actually matter for your app.
