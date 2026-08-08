# Q1. Does WebFlux Guarantee Better Performance, Automatically?

## Simple Explanation (Think of a Sports Car That Still Needs Its Handbrake Released)

A WebFlux app that "looks reactive" can still perform *worse* than Spring MVC if a
single hidden blocking call is left on an event-loop thread — like a sports car
that still won't move because the handbrake is on. This topic is the final
checkpoint: verifying the promised scalability actually materializes.

```
Spring MVC (200 threads, 50ms downstream call):  throughput ceiling ≈ 4,000 req/sec
Spring WebFlux (8-16 threads, same call):         throughput scales MUCH higher,
                                                    since threads never sit idle waiting
                                                    (ONLY IF every layer is truly non-blocking!)
```

---

## Q2. What's the Non-Negotiable Condition for This to Actually Work?

**Every layer must stay genuinely non-blocking** — controller, service,
repository (R2DBC, not JDBC), and external calls (WebClient, not `RestTemplate`).
One hidden blocking call anywhere undoes the benefit for **every request** sharing
that thread.

```
✅ Controller returns Mono/Flux
✅ Service composes reactively
✅ Repository uses R2DBC (not JPA)
✅ External calls use WebClient (not RestTemplate)
     │
     ▼  ALL FOUR must hold true, or the "reactive" label is only partially true
```

---

## Q3. What's the Concrete Resource Payoff?

```
10,000 concurrent slow requests:

Blocking:   needs up to 10,000 threads -> gigabytes wasted on idle stacks
Reactive:   handled by ~8-16 event-loop threads -> memory stays roughly constant
```

Fewer threads → dramatically less memory consumed by thread stacks alone — a real,
measurable infrastructure cost benefit, not just an abstract number.

---

## Q4. How Do I Verify My App Is Actually Behaving Reactively Under Load?

```
Monitor:
  jvm.threads.live                          -> should stay SMALL and STABLE, even under rising load
  reactor.netty.eventloop.pending.tasks      -> tasks waiting for an event-loop thread
```

If thread count climbs proportionally with concurrent load, that's a strong signal
something is secretly blocking somewhere in the pipeline.

---

## Q5. Interview-Style Q&A

### Is WebFlux's scalability benefit the same for CPU-bound workloads?

**No** — reactive doesn't speed up CPU-bound work at all; the advantage is
specific to I/O-bound, high-concurrency scenarios.

### If I see high throughput in a low-traffic test, does that guarantee production performance?

**No** — a hidden blocking call might not show up until real concurrent load
hits it; always load-test, and monitor thread counts specifically.

### What's usually the root cause when a "reactive" app performs no better than its MVC equivalent?

A blocking call (legacy JDBC, `RestTemplate`, `Thread.sleep()`) accidentally left
running on an event-loop thread, undiscovered until traffic increases.

---

## Q6. Summary

```
High-concurrency, I/O-heavy workload?
   │
   ├── YES ──▶ WebFlux's scalability advantage applies strongly
   │              │
   │              ▼
   │        Verify EVERY layer is non-blocking (R2DBC, WebClient, no hidden JDBC/RestTemplate)
   │              │
   │              ▼
   │        Monitor thread count + memory — should stay small & stable under rising load
   │
   └── NO (CPU-bound, low concurrency) ──▶ WebFlux's benefit here is minimal — Spring MVC may be simpler
```

### One sentence to remember

> **"WebFlux's scalability promise only materializes if EVERY layer is
> genuinely non-blocking — verify with monitoring, don't just assume it
> because the code 'looks reactive.'"**
