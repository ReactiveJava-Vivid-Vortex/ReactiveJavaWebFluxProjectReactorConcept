# Flux.generate()

## In Simple Terms

`Flux.generate()` lets you programmatically produce items **one at a time,
synchronously**, in direct response to demand. You're given a `SynchronousSink` and
must call `sink.next(value)` at most once per invocation (or `sink.complete()` /
`sink.error()` to end the stream). It's ideal for generating sequences based on
some internal state (like a counter, or the previous value).

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

Notice `.take(8)` is required — `Flux.generate()` can produce infinitely, so you must
limit it explicitly (or call `sink.complete()` yourself inside the generator function
when done).

## Why It Matters

`Flux.generate()` is inherently **demand-aware** — it only computes the next value
when there's demand for it, making it naturally backpressure-friendly, unlike
manually looping and pushing values with `Flux.create()`. It's the right tool when
your data source is a simple, single-threaded, stateful computation (like a
mathematical sequence or a step-by-step algorithm).
