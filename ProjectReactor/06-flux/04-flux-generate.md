# Flux.generate()

## In Simple Terms

`Flux.generate()` lets you produce items **one at a time**, only when there's
demand for the next one. You get a `SynchronousSink` and call `sink.next(value)`
once per call (or `sink.complete()`/`sink.error()` to end things). It's great for
sequences that depend on some running state, like a counter or the previous
value.

## Simple Example

```java
Flux<Integer> fibonacci = Flux.generate(
    () -> new int[]{0, 1},               // initial state: [previous, current]
    (state, sink) -> {
        sink.next(state[0]);              // emit the "previous" value
        int next = state[0] + state[1];
        state[0] = state[1];
        state[1] = next;
        return state;                      // updated state for next call
    }
);

fibonacci.take(8).subscribe(n -> System.out.print(n + " "));
// Output: 0 1 1 2 3 5 8 13
```

Note the `.take(8)` — without it, `Flux.generate()` could keep going forever, so
you have to limit it yourself (or call `sink.complete()` inside the generator
once you're done).

## Why It Matters

`Flux.generate()` naturally respects demand — it only computes the next value
when something's actually asking for it, so it's automatically safe from
backpressure problems, unlike manually pushing values with `Flux.create()`. Use
it whenever your source is a simple, one-thread-at-a-time, stateful sequence.
