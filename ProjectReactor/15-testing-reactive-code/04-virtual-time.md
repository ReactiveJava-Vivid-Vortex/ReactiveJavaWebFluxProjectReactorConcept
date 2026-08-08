# Virtual Time

## In Simple Terms

"Virtual time" lets you test time-based operators (like `Flux.interval()`
or `.delayElements()`) without actually sitting around and waiting for real
time to pass. `StepVerifier` gives you `.withVirtualTime()`, which
fast-forwards a fake clock instead of forcing your test to literally sleep
for seconds or minutes.

## Simple Example

```java
@Test
void testIntervalWithVirtualTime() {
    StepVerifier.withVirtualTime(() ->
            Flux.interval(Duration.ofHours(1)).take(3)
        )
        .expectSubscription()
        .expectNoEvent(Duration.ofHours(1)) // nothing happens for the first "hour"
        .expectNext(0L)
        .thenAwait(Duration.ofHours(1))
        .expectNext(1L)
        .thenAwait(Duration.ofHours(1))
        .expectNext(2L)
        .verifyComplete();
}
```

Without virtual time, testing `Flux.interval(Duration.ofHours(1))` would
mean literally running the test for 3 hours — not exactly practical. With
virtual time, the same test finishes in milliseconds.

**Good to know:** the `Flux` (or `Mono`) has to be created *inside* the
lambda you pass to `withVirtualTime()` — build it beforehand, and the fake
clock won't be properly wired up before your time-based operators start
using it.

## Why It Matters

Virtual time is what makes it actually practical to test time-dependent
logic — retries with backoff, periodic polling, timeouts — without either
skipping those tests entirely or making your whole test suite painfully
slow.
