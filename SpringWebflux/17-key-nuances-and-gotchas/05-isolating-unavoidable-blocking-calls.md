# You Can Isolate Unavoidable Blocking Calls — WebFlux Doesn't Require 100% Purity

## In Simple Terms

A common (over-corrected) fear after learning "never block an event-loop
thread" is assuming a WebFlux app must have zero blocking calls anywhere,
or it's "not really reactive." That's not quite right. If you genuinely
have no non-blocking alternative (a legacy SDK, a blocking-only vendor
library), you can still use it correctly — by explicitly isolating it on
`Schedulers.boundedElastic()` (see the Threading & Schedulers topic in the
Project Reactor notes), rather than avoiding WebFlux entirely.

## Simple Example

```java
// A legacy, blocking-only library call
LegacySdkResult callLegacySdk() {
    return legacySdk.blockingCall(); // no reactive alternative exists
}

@GetMapping("/legacy-data")
public Mono<ResultDto> getLegacyData() {
    return Mono.fromCallable(this::callLegacySdk)
        .subscribeOn(Schedulers.boundedElastic()) // isolates the blocking call
        .map(ResultMapper::toDto);
}
```

This endpoint isn't as scalable as a fully non-blocking one — under very
high concurrency, `boundedElastic()`'s pool can still become a bottleneck
— but it's correct: the blocking call never steals one of the small,
precious event-loop threads shared by every other request in the app.

## Why It Matters

This nuance matters for realistic migration and integration work: you
rarely control every dependency in a real system, and insisting on "100%
non-blocking or don't use WebFlux at all" is often just not practical. The
real rule (from the Threading & Schedulers topic) is narrower and much
more achievable: never let a blocking call run on an event-loop or
`parallel()` thread — isolate it on `boundedElastic()` instead. A WebFlux
app with a few well-isolated blocking calls is still a legitimate,
correctly-behaving reactive application.
