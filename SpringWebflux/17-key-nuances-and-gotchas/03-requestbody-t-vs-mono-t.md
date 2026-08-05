# @RequestBody T vs @RequestBody Mono&lt;T&gt;

## In Simple Terms

WebFlux lets you declare a controller's request body parameter two ways, and they
behave subtly differently:

```java
@PostMapping
public Mono<OrderDto> create(@RequestBody OrderDto dto) { ... }         // Option A

@PostMapping
public Mono<OrderDto> create(@RequestBody Mono<OrderDto> dtoMono) { ... } // Option B
```

**Option A (`@RequestBody T`)** — WebFlux fully reads and deserializes the request
body internally, *then* calls your method with the ready-made object. From your
method's perspective, the body has already fully arrived.

**Option B (`@RequestBody Mono<T>`)** — your method is invoked immediately, and
*you* receive a `Mono` representing "the body, once it's fully read." You then
`.flatMap()` off of it to continue the reactive chain.

```java
@PostMapping
public Mono<OrderDto> create(@RequestBody Mono<OrderDto> dtoMono) {
    return dtoMono
        .flatMap(dto -> orderService.create(dto))  // chain continues once body arrives
        .map(OrderMapper::toDto);
}
```

## Simple Example

Both ultimately behave equivalently for typical request sizes — the difference
becomes more meaningful for **very large or slow-arriving** request bodies, where
`Mono<T>`'s more explicit, streaming-aware handling can be preferable. For most
everyday CRUD endpoints, plain `@RequestBody T` (Option A) is simpler and just as
correct.

**Remember the validation gotcha from the Validation topic:** `@Valid` reliably
triggers on Option A (`@RequestBody T`), but does **not** reliably auto-trigger on
Option B (`@RequestBody Mono<T>`) — if you use the `Mono<T>` form, validate
explicitly inside your `.flatMap()`.

## Why It Matters

Choosing between these two isn't just style — it affects whether `@Valid`
auto-triggers, and it changes how naturally the body integrates into a fully
reactive chain. Default to plain `@RequestBody T` unless you have a specific reason
(e.g., composing the body directly into a larger reactive pipeline) to use the
`Mono<T>` form.
