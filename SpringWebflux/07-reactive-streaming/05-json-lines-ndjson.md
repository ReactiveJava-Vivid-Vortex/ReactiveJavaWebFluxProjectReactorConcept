# JSON Lines (NDJSON)

## In Simple Terms

**NDJSON** (Newline Delimited JSON, also called JSON Lines) is a format where each
line of the response is a complete, standalone JSON object, separated by newlines —
instead of one giant JSON array wrapping everything. This format is ideal for
streaming, since each line can be parsed and processed independently as it arrives.

## Simple Example

A regular JSON array response (must be fully received before parsing):

```json
[
  {"id": "1", "name": "Widget"},
  {"id": "2", "name": "Gadget"}
]
```

The same data as NDJSON (each line is independently parseable):

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

NDJSON is the standard format for reactive, incremental streaming APIs because it
solves a structural problem: a normal JSON array can't be safely parsed until its
closing `]` bracket arrives, forcing clients to wait for the entire response. NDJSON
lets consumers process each record the instant it arrives, which is essential for
true streaming behavior.
