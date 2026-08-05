# Mono Is Just a Flux of At-Most-One

## In Simple Terms

You don't need two separate mental models for `Mono` and `Flux` — a `Mono<T>` is
simply a `Flux<T>` with an extra promise baked in: **`onNext` will fire at most
once**. Every rule you know about `Flux` (laziness, cold-by-default, the three
signal types, backpressure) applies identically to `Mono` — just with cardinality
capped at 0-or-1 instead of 0-to-N.

This is why so many operators exist on both types with identical names and
behavior (`.map()`, `.filter()`, `.onErrorResume()`, `.doOnNext()`...) — they're the
same underlying idea, just type-parameterized differently.

## Simple Example

```java
// These two lines are conceptually almost the same operation —
// Mono just guarantees "at most 1", Flux allows "0 to N"
Mono<String> oneUser   = userRepository.findById(id);          // 0 or 1 result
Flux<String>  allUsers = userRepository.findAll();              // 0 to N results

// The exact same operators work identically on both:
oneUser.map(String::toUpperCase).subscribe(System.out::println);
allUsers.map(String::toUpperCase).subscribe(System.out::println);
```

You can even convert between them explicitly, which makes the relationship
concrete:

```java
Flux<String> asFlux = oneUser.flux();       // Mono -> Flux (still 0 or 1 item)
Mono<String> firstOnly = allUsers.next();    // Flux -> Mono (takes just the first item)
Mono<List<String>> allAsOne = allUsers.collectList(); // Flux -> Mono (all items, as ONE list)
```

## Why It Matters

Once you see `Mono` as "a `Flux` capped at 1," you stop needing to memorize two
separate rule sets. The only genuinely `Mono`-specific concept is its three-outcome
lifecycle (value, empty, or error — see [[mono-lifecycle]]) — which itself is just
the universal [[the-three-signal-types]] grammar with `onNext` capped at one
instead of unlimited.
