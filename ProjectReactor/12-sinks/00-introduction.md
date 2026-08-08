# Q1. What Is a Sink?

## Simple Explanation (Think of a Microphone at a Radio Station)

Sometimes you need to manually **push** values into a reactive stream from
arbitrary code — a button click, a sensor callback, a background thread — not from
a database query. A `Sinks.Many`/`Sinks.One` is a **microphone**: you hold onto it
and shout values into it from anywhere; a `Flux`/`Mono` on the receiving end lets
subscribers listen.

```
You (anywhere in code)  --[shout into mic]-->  Sink  --[broadcast]-->  Listeners (Flux subscribers)
```

```java
Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

sink.asFlux().subscribe(msg -> System.out.println("Got: " + msg));

sink.tryEmitNext("Hello from anywhere in the code!");
```

---

## Q2. Why Not Just Use the Old `Processor` Interface?

`Sinks` replaced `Processor` because it's much harder to misuse. Every emission
method returns an explicit `EmitResult` you can check, instead of throwing
unpredictable exceptions.

```java
Sinks.EmitResult result = sink.tryEmitNext(42);

if (result.isFailure()) {
    switch (result) {
        case FAIL_OVERFLOW -> System.out.println("Buffer full, item dropped");
        case FAIL_TERMINATED -> System.out.println("Sink already completed");
        default -> System.out.println("Emission failed: " + result);
    }
}
```

---

## Q3. `Sinks.One` vs `Sinks.Many` — Which One Do I Need?

| | `Sinks.One<T>` | `Sinks.Many<T>` |
|---|---|---|
| Emits | Exactly ONE value (like a programmatic `Mono`) | Many values over time (like a programmatic `Flux`) |
| Methods | `tryEmitValue()`, `tryEmitError()` | `tryEmitNext()` (repeatable), `tryEmitComplete()`, `tryEmitError()` |

```java
Sinks.One<String> one = Sinks.one();
one.asMono().subscribe(v -> System.out.println("Got: " + v));
one.tryEmitValue("Only once!"); // can only be called once
```

---

## Q4. Multicast vs Unicast vs Replay — How Do I Choose?

```java
// MULTICAST: broadcasts to ALL current subscribers — like a live radio broadcast
Sinks.Many<Integer> multicast = Sinks.many().multicast().onBackpressureBuffer();

// UNICAST: exactly ONE subscriber allowed — buffers until it shows up
Sinks.Many<Integer> unicast = Sinks.many().unicast().onBackpressureBuffer();

// REPLAY: multicast + remembers history for LATE subscribers too
Sinks.Many<Integer> replay = Sinks.many().replay().limit(2); // remember last 2
```

```
Multicast + subscriber joins LATE  -> misses everything emitted before joining
Replay + subscriber joins LATE     -> catches up on recent history first, then goes live
Unicast + a SECOND subscriber tries to join -> typically errors — only 1 allowed
```

---

## Q5. What Does "Direct Best Effort" Mean?

No buffering at all — if a subscriber isn't currently ready (no outstanding
demand), the emission is simply **dropped** for that subscriber.

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().directBestEffort();
```

This trades guaranteed delivery for bounded memory — appropriate when the
**latest** value matters more than never missing one (e.g., live sensor readings).

---

## Q6. How Do I Build a Simple Event Bus with Sinks?

```java
public class OrderEventBus {
    private final Sinks.Many<OrderEvent> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void publish(OrderEvent event) {
        sink.tryEmitNext(event);
    }

    public Flux<OrderEvent> subscribeToEvents() {
        return sink.asFlux();
    }
}

bus.subscribeToEvents().subscribe(event -> emailService.notify(event));
bus.subscribeToEvents().subscribe(event -> auditLogger.log(event));
bus.publish(new OrderEvent("ORD-123", "CREATED")); // BOTH listeners react
```

---

## Q7. Interview-Style Q&A

### Is a `Sinks.Many` a hot or cold publisher?

**Hot.** It produces data independently of subscribers — exactly the mechanism
used to build hot publishers manually (see the Hot & Cold Publishers topic).

### What happens if I call `tryEmitNext()` after `tryEmitComplete()`?

It fails — the returned `EmitResult` will be `FAIL_TERMINATED`. Always check the
result rather than assuming success.

### Should I use `tryEmitNext()` or `emitNext()`?

`tryEmitNext()` returns an `EmitResult` you check yourself; `emitNext()` takes an
`EmitFailureHandler` and can throw. Prefer `tryEmitNext()` for explicit,
predictable handling.

---

## Q8. Summary

| Concept | Key Takeaway |
|---|---|
| Sinks.One | Programmatic single-value producer (like a manual Mono) |
| Sinks.Many | Programmatic multi-value producer (like a manual Flux) |
| Multicast | Broadcasts to current subscribers only — late joiners miss history |
| Unicast | Exactly one subscriber, buffered until it arrives |
| Replay | Multicast + remembers history for late subscribers |
| Producer API | `tryEmitXxx()` returns an `EmitResult` — always check it, never ignore |

### One sentence to remember

> **"A Sink is a microphone you hold — shout a value in from anywhere in your
> code, and every subscriber listening on the other end hears it, live."**
