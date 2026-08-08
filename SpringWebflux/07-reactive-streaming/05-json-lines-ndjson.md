# JSON Lines (NDJSON)

## In Simple Terms

NDJSON (Newline Delimited JSON, also called JSON Lines) is a format where
each line of the response is its own complete, standalone JSON object,
separated by newlines — instead of one big JSON array wrapping everything
together. This is perfect for streaming, since each line can be read and
handled on its own the moment it arrives.

## Simple Example

A regular JSON array response (has to be fully received before it can even
be parsed):

```json
[
  {"id": "1", "name": "Widget"},
  {"id": "2", "name": "Gadget"}
]
```

The same data as NDJSON (each line stands on its own):

```
{"id": "1", "name": "Widget"}
{"id": "2", "name": "Gadget"}
```

Producing NDJSON in WebFlux:

```java
@GetMapping(value = "/products", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> streamProducts() {
    return productRepository.findAll().map(ProductMapper::toDto);
}
```

## Why It Matters

NDJSON is the go-to format for reactive, incremental streaming APIs
because it fixes a real structural problem: a normal JSON array can't be
safely parsed until its closing `]` shows up, forcing clients to wait for
the whole thing. NDJSON lets consumers process each record the instant it
arrives, which is exactly what true streaming needs.
