# Lazy Execution

## In Simple Terms

Reactive pipelines are **lazy** — writing `.map()`, `.filter()`, and so on
doesn't actually run any of that code. It just builds a description of what
*should* happen later. Nothing actually runs until `.subscribe()` gets called —
either by you directly, or by a framework like Spring WebFlux handling an HTTP
request.

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

Notice "Supplier is running!" only shows up **after** `.subscribe()` — not when
we defined `Mono.fromSupplier(...)`.

## Why It Matters

This is one of the biggest sources of "why isn't my code doing anything?!"
confusion for beginners — it's almost always because `.subscribe()` was never
called (or the framework never got the chance to, because the `Mono`/`Flux` was
returned and ignored somewhere). Once laziness clicks, powerful tricks like
`Mono.defer()` — which re-runs its logic fresh for every subscriber — start
making sense too.
