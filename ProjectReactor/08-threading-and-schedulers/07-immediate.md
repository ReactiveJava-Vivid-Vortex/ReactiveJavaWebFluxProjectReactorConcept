# immediate()

## In Simple Terms

`Schedulers.immediate()` is a special "no-op" scheduler that runs tasks
**synchronously, on the current thread**, without any actual scheduling or thread
switching. It exists mainly as a default/no-op placeholder value for APIs that
require a `Scheduler` parameter, or for testing.

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

Notice nothing actually changed — it behaves as if `.publishOn()` was never called.

## Why It Matters

`Schedulers.immediate()` is rarely used directly in application code, but it's useful
as a default value in configurable APIs (e.g., a library that lets you optionally
specify a `Scheduler`, defaulting to `immediate()` meaning "don't switch threads
unless told to"), or in tests where you want deterministic, synchronous execution
without any real concurrency involved.
