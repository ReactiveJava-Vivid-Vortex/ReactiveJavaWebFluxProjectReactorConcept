# Mono Is Just a Flux of At-Most-One

## In Simple Terms

You really don't need two separate mental models for `Mono` and `Flux` — a
`Mono<T>` is just a `Flux<T>` with one extra promise attached: `onNext`
will fire **at most once**. Everything you already know about `Flux` —
laziness, being cold by default, the three signal types, backpressure —
applies to `Mono` in exactly the same way, just with the count capped at
0-or-1 instead of 0-to-many.

That's why so many operators exist on both types with identical names and
behavior (`.map()`, `.filter()`, `.onErrorResume()`, `.doOnNext()`...) —
they're the exact same idea, just wearing a different type parameter.

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

You can even convert between them directly, which makes the relationship
concrete:

```java
Flux<String> asFlux = oneUser.flux();       // Mono -> Flux (still 0 or 1 item)
Mono<String> firstOnly = allUsers.next();    // Flux -> Mono (takes just the first item)
Mono<List<String>> allAsOne = allUsers.collectList(); // Flux -> Mono (all items, as ONE list)
```

## Why It Matters

Once you see `Mono` as simply "a `Flux` capped at 1," you stop needing to
memorize two separate rulebooks. The only genuinely `Mono`-specific idea is
its three-outcome lifecycle (value, empty, or error — see
[[mono-lifecycle]]) — and even that's just the universal
[[the-three-signal-types]] pattern, with `onNext` capped at one instead of
unlimited.
