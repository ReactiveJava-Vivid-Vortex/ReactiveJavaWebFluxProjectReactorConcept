# take()

## In Simple Terms

`.take(n)` says "give me the first `n` items, and then stop" — like telling
someone dealing cards "just give me 3 and you can put the rest away." Once
it has its `n` items, it cancels the source and wraps up, even if there was
much more available. There's also a time-based version, `.take(Duration)`,
that grabs whatever shows up within a time window instead of a fixed count.

## Simple Example

```java
Flux.range(1, 100)
    .take(3)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 1
Got: 2
Got: 3
```

Even though the source could give you 100 items, `.take(3)` stops after just
3 — the other 97 are never even produced, as long as the source respects the
"I only want a few" signal.

Time-based version:

```java
Flux.interval(Duration.ofMillis(100))
    .take(Duration.ofSeconds(1)) // take whatever comes in over 1 second
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

## Why It Matters

`.take()` is your go-to for putting a lid on streams that could otherwise
run forever (like `Flux.interval()`), and it's the easiest way to write
short, predictable tests and demos without manually cancelling anything
yourself.
