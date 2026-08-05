# Flux.fromIterable()

## In Simple Terms

`Flux.fromIterable(iterable)` creates a `Flux` that emits each element of an existing
`Iterable` (like a `List`, `Set`, etc.), one at a time, in order. This is the most
common way to turn regular Java collections into a reactive stream.

## Simple Example

```java
List<String> names = List.of("Alice", "Bob", "Charlie");

Flux.fromIterable(names)
    .map(String::toUpperCase)
    .subscribe(name -> System.out.println("Name: " + name));
```

Output:
```
Name: ALICE
Name: BOB
Name: CHARLIE
```

A very common real-world pattern: converting a database result (already fetched as a
`List`) into a `Flux` for further reactive processing:

```java
List<Order> orders = orderRepository.findAllBlocking(); // hypothetical blocking call
Flux<Order> orderFlux = Flux.fromIterable(orders);
```

(Note: in real reactive code, you'd typically get a `Flux<Order>` directly from a
reactive repository, rather than fetching a blocking `List` first.)

## Why It Matters

`Flux.fromIterable()` is the standard bridge between "regular Java collections" and
"reactive streams" — extremely useful when you have data already in memory (e.g.,
static configuration, a small in-memory cache) that you want to process using
Reactor's operators.
