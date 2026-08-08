# map()

## In Simple Terms

`.map()` takes each item as it comes through and turns it into something else —
right there, on the spot, no waiting around. Think of it like a factory
conveyor belt where a worker picks up each box, changes what's inside, and
puts it back on the belt. One box in, one box out, every time.

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

A very common real use: turning a database entity into something you can
send back to a client.

```java
Flux<Order> orders = orderRepository.findAll();

Flux<OrderDto> dtos = orders.map(order -> new OrderDto(order.getId(), order.getTotal()));
```

**One rule to remember:** the worker on the belt must do the change
instantly — no going off to fetch something first. If turning one item into
the next requires waiting on something else (a database call, a network
request), that's a job for `.flatMap()`, not `.map()`.

## Why It Matters

You'll reach for `.map()` constantly. Anytime you just need to reshape a
value — entity to DTO, number to formatted text, raw data to something more
useful — and there's no waiting involved, `.map()` is the tool.
