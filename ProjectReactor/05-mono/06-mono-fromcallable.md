# Mono.fromCallable()

## In Simple Terms

`Mono.fromCallable(callable)` is basically the same as `Mono.fromSupplier()`,
except it's for code that **might throw a checked exception** — like reading a
file. It's lazy (runs only when subscribed), and if the code throws, that
exception is automatically turned into an `onError()` signal instead of blowing
up your program.

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

If `config.txt` doesn't exist, the `IOException` gets caught automatically and
turned into an `onError()` signal — no manual try/catch needed on your end.

## Why It Matters

`Mono.fromCallable()` is the tool for wrapping any **blocking, synchronous** call
that might throw a checked exception — like file I/O or old-style JDBC code —
into the reactive world. It's often paired with
`.subscribeOn(Schedulers.boundedElastic())` to make sure that blocking call runs
on a thread pool built for blocking work, not one of the few limited event-loop
threads.

```java
Mono.fromCallable(() -> legacyBlockingJdbcCall())
    .subscribeOn(Schedulers.boundedElastic())
    .subscribe(result -> System.out.println(result));
```
