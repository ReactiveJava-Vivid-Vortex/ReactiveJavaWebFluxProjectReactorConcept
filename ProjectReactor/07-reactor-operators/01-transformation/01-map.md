# map()

## In Simple Terms

`.map(fn)` transforms each item emitted by a `Mono`/`Flux` into a **different value**,
synchronously, one-to-one. It's the reactive equivalent of `Stream.map()` in the Java
Streams API — for every input item, you get exactly one output item.

## Simple Example

```java
Flux.just(1, 2, 3, 4)
    .map(n -> n * n)
    .subscribe(square -> System.out.println("Square: " + square));
```

Output:
```
Square: 1
Square: 4
Square: 9
Square: 16
```

Common usage: converting entities to DTOs.

```java
Flux<Order> orders = orderRepository.findAll();

Flux<OrderDto> dtos = orders.map(order -> new OrderDto(order.getId(), order.getTotal()));
```

**Important:** `.map()` must be **synchronous** — if your transformation itself
returns a `Mono`/`Flux` (i.e., it's asynchronous), use `.flatMap()` instead, not
`.map()`.

## Why It Matters

`.map()` is probably the single most-used operator in reactive pipelines — any time
you need to reshape data (entity → DTO, raw value → formatted string, etc.) without
performing any additional asynchronous work, `.map()` is the right tool.
