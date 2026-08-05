# Virtual Time

## In Simple Terms

"Virtual time" lets you test time-based operators (like `Flux.interval()`,
`.delayElements()`) **without actually waiting in real time**. `StepVerifier`
provides `.withVirtualTime()`, which fast-forwards a simulated clock instead of
making your test suite slow (or flaky) by literally sleeping for seconds or minutes.

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

Without virtual time, testing `Flux.interval(Duration.ofHours(1))` would require the
test to literally run for 3 hours — completely impractical. With virtual time, this
test completes in milliseconds.

**Important:** the `Flux` (or `Mono`) must be created **inside** the lambda passed to
`withVirtualTime()` — if it's created beforehand, the virtual clock won't be properly
installed before the source starts using time-based operators.

## Why It Matters

Virtual time is what makes it *practical* to properly test time-dependent reactive
logic — retries with backoff, periodic polling, timeouts — without either skipping
those tests entirely or making your test suite unbearably slow.
