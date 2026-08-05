# When to Use Reactive Programming

## In Simple Terms

Reactive programming is a **great fit** when your application:

- Handles a **high volume of concurrent requests**.
- Spends most of its time **waiting on I/O** (databases, external HTTP calls, message
  queues) rather than doing heavy CPU computation.
- Needs to be **resilient and elastic** under unpredictable load spikes.
- Benefits from **streaming** data (e.g., large files, live event feeds) instead of
  loading everything into memory at once.

It is a **poor fit** when your application:

- Is mostly **CPU-bound** (heavy number crunching, image processing) rather than
  I/O-bound — reactive doesn't make CPU-bound work faster.
- Has **low concurrency** needs (a small internal admin tool used by 5 people).
- Needs simple, straightforward code and the team is not experienced with reactive
  debugging/testing (there's a real learning-curve cost).

## Simple Example

```
Use Case                                   Good Fit for Reactive?
------------------------------------------ -----------------------
Public API gateway, 50k req/sec            Yes — I/O heavy, high concurrency
Streaming stock prices to browsers (SSE)   Yes — natural streaming fit
Batch job resizing 10,000 images (CPU)     No — CPU-bound, reactive adds no benefit
Small internal CRUD tool, 10 users         No — added complexity isn't worth it
Microservice aggregating 5 downstream APIs Yes — parallel non-blocking calls shine
```

## Why It Matters

Choosing reactive programming isn't "always better" — it's a trade-off. You gain
scalability and resilience under I/O-heavy load, at the cost of a steeper learning
curve, trickier debugging (stack traces look different), and more complex testing.
Use it where its strengths actually apply.
