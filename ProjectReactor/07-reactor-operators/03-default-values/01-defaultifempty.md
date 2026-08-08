# defaultIfEmpty()

## In Simple Terms

`.defaultIfEmpty(value)` says "if nothing shows up, just use this instead."
If the stream finishes without ever emitting a single item, it hands out a
fixed, ready-made backup value. If the stream did produce something, the
backup is never used at all.

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

`.defaultIfEmpty()` is the simplest way to avoid an awkward "nothing
happened" outcome when you have an obvious fallback — showing "No results"
text, or falling back to a standard setting when nothing was configured.
