# Lazy Execution

## In Simple Terms

Reactive pipelines in Project Reactor are **lazy** — building a `Mono` or `Flux` (with
`.map()`, `.filter()`, etc.) does **not** run any code. It only builds a description
of *what should happen*. The actual execution only starts once you call
`.subscribe()` (directly, or indirectly via a framework like Spring WebFlux, which
subscribes for you when handling an HTTP request).

## Simple Example

```java
public class LazyDemo {
    public static void main(String[] args) {
        System.out.println("Before building pipeline");

        Mono<String> mono = Mono.fromSupplier(() -> {
            System.out.println("Supplier is running!"); // won't print yet
            return "Hello";
        });

        System.out.println("Pipeline built, but nothing ran yet");

        mono.subscribe(value -> System.out.println("Got value: " + value));
    }
}
```

Output:
```
Before building pipeline
Pipeline built, but nothing ran yet
Supplier is running!
Got value: Hello
```

Notice "Supplier is running!" only prints **after** `.subscribe()` is called — not
when `Mono.fromSupplier(...)` was defined.

## Why It Matters

This is one of the most common sources of confusion for beginners: **"why doesn't my
reactive code do anything?"** — usually because `.subscribe()` was never called (or
the framework never got a chance to subscribe, e.g., because the `Mono`/`Flux` return
value was ignored). Understanding laziness also enables powerful patterns like
`Mono.defer()`, which re-evaluates the supplier logic fresh for every subscriber.
