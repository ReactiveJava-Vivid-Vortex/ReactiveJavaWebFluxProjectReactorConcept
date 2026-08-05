# Responsive

## In Simple Terms

"Responsive" (from the Reactive Manifesto) means a system provides timely responses,
consistently, whenever possible — not just fast on average, but with **predictable,
bounded** response times, even under load or when problems occur. A responsive
system fails fast and clearly, rather than hanging indefinitely.

## Simple Example

```java
@GetMapping("/product/{id}")
public Mono<Product> getProduct(@PathVariable String id) {
    return productService.findById(id)
        .timeout(Duration.ofSeconds(2)) // bounded — never hangs indefinitely
        .onErrorResume(TimeoutException.class, e ->
            Mono.just(Product.unavailablePlaceholder())
        );
}
```

Even if the underlying service is slow or unresponsive, this endpoint guarantees a
response within roughly 2 seconds — either the real product, or a clear
"unavailable" placeholder — rather than leaving the client waiting indefinitely.

## Why It Matters

Responsiveness is what users and downstream systems actually experience — a system
that's "usually fast but occasionally hangs forever" is far worse in practice than
one that's consistently a bit slower but always responds within a known bound.
Reactive programming's tools (`timeout()`, `onErrorResume()`) make building
genuinely responsive systems straightforward.
