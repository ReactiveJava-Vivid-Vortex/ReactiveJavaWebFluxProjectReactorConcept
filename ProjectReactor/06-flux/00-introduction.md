# Q1. What Is a Flux?

## Simple Explanation (Think of a Playlist vs a Live Radio)

If `Mono` is a vending machine slot (0 or 1 item), `Flux<T>` is a **playlist** —
0, 1, or even infinitely many items, delivered one at a time, over time.

```
Mono:  🎵           (0 or 1 song)
Flux:  🎵 🎵 🎵 🎵 ... (0 to N songs, could be a whole playlist, could be a live stream)
```

Everything you know about `Mono` (laziness, cold-by-default, the three signal
types) applies to `Flux` too — just without the "at most one" cap on `onNext`.

---

## Q2. How Do I Create a Flux? (Pick Based on Your Source)

| Source Shape | Factory | Example |
|---|---|---|
| Already have fixed values | `Flux.just()` | `Flux.just("a", "b", "c")` |
| A sequence of numbers | `Flux.range()` | `Flux.range(1, 5)` |
| An existing collection | `Flux.fromIterable()` | `Flux.fromIterable(myList)` |
| Generate one at a time, synchronously | `Flux.generate()` | Fibonacci sequence, stateful counters |
| External async/push source | `Flux.create()` / `Flux.push()` | Message listeners, sensors |
| A repeating timer | `Flux.interval()` | Heartbeats, polling |

```java
Flux.range(1, 5)
    .map(n -> n * n)
    .subscribe(square -> System.out.println("Square: " + square));
// Square: 1, 4, 9, 16, 25
```

---

## Q3. `Flux.generate()` vs `Flux.create()` — What's the Real Difference?

```java
// generate(): ONE item per callback invocation, synchronous, naturally demand-aware
Flux<Integer> generated = Flux.generate(sink -> {
    sink.next((int) (Math.random() * 100)); // exactly one item, respects backpressure automatically
});

// create(): can push MANY items per callback, from ANY thread, async sources
Flux<String> created = Flux.create(sink -> {
    externalEventSource.onEvent(event -> sink.next(event)); // called from any thread, any time
    externalEventSource.onClose(sink::complete);
});
```

| | `generate()` | `create()` / `push()` |
|---|---|---|
| Emission | One item per call, synchronous | Many items, can be async/multi-threaded |
| Backpressure | Automatic (demand-aware) | You must configure an `OverflowStrategy` |
| Best for | Stateful sequences (Fibonacci, counters) | Bridging external push-based sources |

---

## Q4. Finite vs Infinite Streams — Why Does It Matter?

```
Flux.range(1, 5)          -> FINITE (completes after 5 items, on its own)
Flux.fromIterable(list)   -> FINITE (completes after the list is exhausted)
Flux.interval(Duration)   -> INFINITE (never completes on its own!)
```

```java
Flux.interval(Duration.ofSeconds(1))
    .take(5) // MUST bound an infinite stream explicitly, or it runs forever
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

**Operators that don't work on infinite streams:** `.collectList()`, `.count()`,
`.reduce()` — they all need `onComplete()` to ever fire, which never happens on a
truly infinite `Flux`.

---

## Q5. What Is `FluxSink`?

The manual emission "microphone" inside `Flux.create()`/`Flux.push()` — call
`.next(item)` many times, then `.complete()` or `.error()` once.

```java
Flux<Integer> flux = Flux.create(sink -> {
    for (int i = 1; i <= 5; i++) sink.next(i);
    sink.complete();
});
```

Unlike `MonoSink` (one emission only), `FluxSink` supports repeated `.next()`
calls — and needs an `OverflowStrategy` (`BUFFER`, `DROP`, `LATEST`, `ERROR`) for
when the producer outpaces demand.

---

## Q6. Interview-Style Q&A

### Does `Flux.just("a", "b", "c")` re-run for every subscriber?

**Yes** — it's cold by default, like most Reactor sources. Each subscription gets
a fresh, independent emission of `"a", "b", "c"`.

### If I don't call `.take(n)` on `Flux.interval()`, what happens?

It runs **forever**, ticking indefinitely, unless externally cancelled — this is a
very common beginner bug in demos/tests.

### Can `Flux.generate()` push more than one item per invocation?

**No** — that's the key distinction from `Flux.create()`. `generate()` is strictly
one item per call, which is what makes it automatically backpressure-safe.

### What's the simplest way to turn a `List<T>` into a Flux?

`Flux.fromIterable(myList)`.

---

## Q7. Summary

| Concept | Key Takeaway |
|---|---|
| Flux | 0 to N async values — the "big sibling" of Mono, same rules, no upper cap |
| Flux.generate() | Synchronous, one-at-a-time, automatically demand-aware |
| Flux.create()/push() | Async/push-based bridge, needs an overflow strategy |
| Finite vs Infinite | Infinite streams (`interval()`) must be explicitly bounded (`.take()`) |
| FluxSink | Manual "microphone" for emitting many items into a Flux.create() |

### One sentence to remember

> **"A Flux is a playlist, not a jukebox slot — it can hand you nothing, one
> song, or an endless live stream, and you decide how many tracks you're ready
> to hear next."**
