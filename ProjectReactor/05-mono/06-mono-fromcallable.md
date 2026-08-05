# Mono.fromCallable()

## In Simple Terms

`Mono.fromCallable(callable)` is like `Mono.fromSupplier()`, but for code that **can
throw a checked exception** (a `Callable<T>` instead of a `Supplier<T>`). It's lazy
(runs only on subscription) and automatically converts any thrown exception into an
`onError()` signal instead of propagating it as a raw exception.

## Simple Example

```java
Mono<String> mono = Mono.fromCallable(() -> {
    // Files.readString() throws a checked IOException
    return Files.readString(Path.of("config.txt"));
});

mono.subscribe(
    content -> System.out.println("File content: " + content),
    error -> System.out.println("Failed to read file: " + error.getMessage())
);
```

If `config.txt` doesn't exist, the `IOException` is automatically caught and turned
into an `onError()` signal — you never need a manual `try/catch` around it.

## Why It Matters

`Mono.fromCallable()` is the go-to wrapper for any **blocking, synchronous** call that
might throw a checked exception — like file I/O or legacy JDBC code — that you need
to bridge into a reactive pipeline. It's very commonly combined with
`.subscribeOn(Schedulers.boundedElastic())` to make sure that blocking call runs on a
thread pool designed for blocking work, not on a limited event-loop thread.

```java
Mono.fromCallable(() -> legacyBlockingJdbcCall())
    .subscribeOn(Schedulers.boundedElastic())
    .subscribe(result -> System.out.println(result));
```
