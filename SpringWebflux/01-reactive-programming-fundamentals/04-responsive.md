# Responsive

## In Simple Terms

"Responsive" means a system gives timely answers, consistently — not just
fast on a good day, but with predictable, bounded response times, even
under load or when something's going wrong. A responsive system fails fast
and clearly instead of just hanging forever.

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

Even if the underlying service is slow or stuck, this endpoint guarantees
an answer within roughly 2 seconds — either the real product, or a clear
"unavailable" placeholder — instead of leaving the client hanging
indefinitely.

## Why It Matters

Responsiveness is what people actually feel when using a system — one
that's "usually fast but occasionally hangs forever" is much worse in
practice than one that's a bit slower but always answers within a known
window. Tools like `timeout()` and `onErrorResume()` make building a
genuinely responsive system straightforward.
