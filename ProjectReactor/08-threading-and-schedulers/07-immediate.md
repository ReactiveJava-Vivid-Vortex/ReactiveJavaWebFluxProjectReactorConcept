# immediate()

## In Simple Terms

`Schedulers.immediate()` is a "do nothing special" scheduler — it just runs
tasks right where you are, on the current thread, with no actual switching
involved. It mostly exists as a placeholder for APIs that expect a
`Scheduler` argument but don't actually need one, or for tests.

## Simple Example

```java
Mono.just("Hello")
    .publishOn(Schedulers.immediate()) // effectively a no-op — stays on the same thread
    .subscribe(value -> System.out.println("Running on: " + Thread.currentThread().getName()));
```

Output:
```
Running on: main
```

Notice nothing changed — it behaves exactly as if `.publishOn()` was never
called at all.

## Why It Matters

You'll rarely reach for `Schedulers.immediate()` directly in application
code, but it's a useful default in configurable APIs (a library that lets
you optionally supply a `Scheduler`, defaulting to "don't switch threads
unless told to"), or in tests where you want predictable, synchronous
behavior without any real concurrency getting in the way.
