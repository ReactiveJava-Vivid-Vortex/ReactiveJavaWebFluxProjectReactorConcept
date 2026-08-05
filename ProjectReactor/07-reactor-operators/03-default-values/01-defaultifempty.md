# defaultIfEmpty()

## In Simple Terms

`.defaultIfEmpty(value)` provides a fallback **single, fixed value** to emit if the
upstream `Mono`/`Flux` completes without emitting anything at all. If the upstream
does emit at least one item, the default is never used.

## Simple Example

```java
Mono<String> userName = findUserName("unknown-id"); // returns Mono.empty()

userName
    .defaultIfEmpty("Guest")
    .subscribe(name -> System.out.println("Welcome, " + name));
// Output: Welcome, Guest
```

With a `Flux`:

```java
Flux.<String>empty()
    .defaultIfEmpty("No items found")
    .subscribe(System.out::println);
// Output: No items found

Flux.just("a", "b")
    .defaultIfEmpty("No items found") // never used, since the Flux is not empty
    .subscribe(System.out::println);
// Output: a
//         b
```

## Why It Matters

`.defaultIfEmpty()` is the simplest way to avoid "nothing happened" outcomes in your
pipeline when a sensible static fallback exists — e.g., showing "No results" text, or
defaulting to a standard configuration value when none is set.
