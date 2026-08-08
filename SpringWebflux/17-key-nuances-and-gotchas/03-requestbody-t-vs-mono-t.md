# @RequestBody T vs @RequestBody Mono&lt;T&gt;

## In Simple Terms

WebFlux lets you declare a controller's request body two different ways,
and they behave a bit differently:

```java
@PostMapping
public Mono<OrderDto> create(@RequestBody OrderDto dto) { ... }         // Option A

@PostMapping
public Mono<OrderDto> create(@RequestBody Mono<OrderDto> dtoMono) { ... } // Option B
```

**Option A (`@RequestBody T`)** — WebFlux fully reads and parses the
request body internally first, *then* calls your method with the finished
object. From your method's point of view, the body has already fully
arrived.

**Option B (`@RequestBody Mono<T>`)** — your method gets called right
away, and *you* get handed a `Mono` representing "the body, once it's
fully read." You then `.flatMap()` off of it to keep the chain going.

```java
@PostMapping
public Mono<OrderDto> create(@RequestBody Mono<OrderDto> dtoMono) {
    return dtoMono
        .flatMap(dto -> orderService.create(dto))  // chain continues once body arrives
        .map(OrderMapper::toDto);
}
```

## Simple Example

For typical request sizes, both behave more or less the same — the
difference matters more for very large or slow-to-arrive request bodies,
where `Mono<T>`'s more explicit, streaming-friendly handling can help. For
most everyday CRUD endpoints, plain `@RequestBody T` (Option A) is simpler
and just as correct.

**Remember the validation gotcha from the Validation topic:** `@Valid`
reliably triggers with Option A (`@RequestBody T`), but doesn't reliably
auto-trigger with Option B (`@RequestBody Mono<T>`) — if you use the
`Mono<T>` form, validate explicitly inside your `.flatMap()`.

## Why It Matters

Picking between these isn't just a style preference — it affects whether
`@Valid` fires automatically, and it changes how naturally the body flows
into a fully reactive chain. Default to plain `@RequestBody T` unless you
have a specific reason (like composing the body directly into a bigger
reactive pipeline) to reach for the `Mono<T>` form.
