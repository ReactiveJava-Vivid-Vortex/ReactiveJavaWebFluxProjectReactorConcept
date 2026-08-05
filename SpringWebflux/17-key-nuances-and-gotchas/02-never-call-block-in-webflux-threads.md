# Never Call .block() on a WebFlux Request Thread

## In Simple Terms

`Mono`/`Flux` have a `.block()` method that synchronously waits for and returns the
value — tempting when you just want "the value, right now," especially if you're
new to reactive code. **Never call `.block()` (or `.blockFirst()`/`.blockLast()`)
inside code that's running on a WebFlux request-handling thread** (a controller, a
filter, or anything invoked as part of handling an HTTP request).

Doing so doesn't just "work but slowly" — Reactor actively **detects** this in many
cases and throws an exception specifically to stop you:

```
java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking,
which is not supported in thread reactor-http-nio-2
```

## Simple Example

```java
// BAD: blocks an event-loop thread — may throw IllegalStateException,
// and even if it doesn't, it defeats the entire purpose of WebFlux
@GetMapping("/products/{id}")
public ProductDto getProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .block(); // <-- DON'T DO THIS in a WebFlux controller
}

// GOOD: return the Mono itself, let WebFlux subscribe to it non-blockingly
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
}
```

`.block()` is only acceptable in genuinely non-reactive contexts completely
outside the WebFlux request-handling pipeline — e.g., a `main()` method in a
one-off standalone script, or in test setup code.

## Why It Matters

WebFlux's entire scalability story depends on its small pool of event-loop threads
never freezing. A single `.block()` call inside a controller does exactly that —
it holds one of those precious threads hostage until the blocked call finishes,
directly undermining the reason you chose WebFlux in the first place. If you ever
feel like you need `.block()` inside request-handling code, it's a signal that
something upstream should have returned a `Mono`/`Flux` instead of a plain value.
