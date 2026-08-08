# Never Call .block() on a WebFlux Request Thread

## In Simple Terms

`Mono`/`Flux` have a `.block()` method that just freezes and waits for the
value — tempting when you want "the value, right now," especially if
you're new to reactive code. Never call `.block()` (or
`.blockFirst()`/`.blockLast()`) inside code running on a WebFlux
request-handling thread (a controller, a filter, anything invoked while
handling an HTTP request).

Doing so doesn't just "work but slowly" — Reactor actually catches this in
many cases and throws an exception specifically to stop you:

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

`.block()` is only okay in genuinely non-reactive contexts completely
outside the WebFlux request path — like a one-off `main()` method in a
standalone script, or in test setup code.

## Why It Matters

WebFlux's entire scalability story rests on its small pool of event-loop
threads never freezing. A single `.block()` call inside a controller does
exactly that — it holds one of those precious threads hostage until the
blocked call finishes, undercutting the whole reason you picked WebFlux in
the first place. If you ever feel like you need `.block()` inside
request-handling code, that's a sign something upstream should have
returned a `Mono`/`Flux` instead of a plain value.
