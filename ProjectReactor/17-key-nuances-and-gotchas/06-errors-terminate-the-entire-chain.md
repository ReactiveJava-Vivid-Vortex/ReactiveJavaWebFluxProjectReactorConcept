# Errors Terminate the Entire Chain

## In Simple Terms

Per the [[the-three-signal-types]] rules, `onError` ends everything — once
it fires, the stream is over, full stop. There's no built-in way to "catch
an error and keep the same stream running" the way a `try/catch` inside a
`for` loop lets you skip one bad iteration and keep looping. If processing
a single item throws, the *whole* `Flux` ends right there — every item
still left, no matter how many, never even gets attempted.

This trips up a lot of people coming from regular imperative code, where a
`try/catch` inside a loop body naturally lets the loop keep going.

## Simple Example

```java
Flux.just(1, 2, 0, 4, 5)
    .map(n -> 10 / n) // throws on n == 0
    .subscribe(
        v -> System.out.println("Got: " + v),
        e -> System.out.println("Stream ended with error: " + e.getMessage())
    );

// Output:
// Got: 10
// Got: 5
// Stream ended with error: / by zero
// (4 and 5 are NEVER processed — the whole stream stopped, not just that one item!)
```

If you actually want "skip the bad item, keep going" behavior (closer to
what `try/catch` inside a loop gives you), handle the error *per item* —
usually inside a `flatMap`, so a failure only ends that one inner `Mono`,
not the outer `Flux`:

```java
Flux.just(1, 2, 0, 4, 5)
    .flatMap(n -> Mono.fromCallable(() -> 10 / n)
        .onErrorResume(e -> {
            System.out.println("Skipping bad item: " + n);
            return Mono.empty(); // this ONE item is skipped; outer Flux keeps going
        })
    )
    .subscribe(v -> System.out.println("Got: " + v));

// Output:
// Got: 10
// Got: 5
// Skipping bad item: 0
// Got: 2   (order may vary — flatMap runs inner Monos concurrently by default)
// Got: 2
```

## Why It Matters

Forgetting this rule is a common source of "why did my batch job stop
halfway through?!" bugs — one bad record in a large `Flux` can silently cut
off processing for everything after it, unless you deliberately isolate
each item's error handling (usually `flatMap` plus
`onErrorResume`/`onErrorContinue`) so one failure doesn't take the whole
stream down with it.
