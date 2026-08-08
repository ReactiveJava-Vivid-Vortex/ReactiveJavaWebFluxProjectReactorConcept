# Flux.fromIterable()

## In Simple Terms

`Flux.fromIterable(iterable)` takes an existing `List`, `Set`, or any
`Iterable`, and sends out each element, one at a time. It's the go-to way to
turn a normal Java collection into a reactive stream.

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

A common pattern — turning an already-fetched list into a `Flux` for further
processing:

```java
List<Order> orders = orderRepository.findAllBlocking(); // hypothetical blocking call
Flux<Order> orderFlux = Flux.fromIterable(orders);
```

(In real reactive code, you'd usually get a `Flux<Order>` directly from a
reactive repository, rather than fetching a plain `List` first.)

## Why It Matters

`Flux.fromIterable()` is the standard bridge between "a regular Java collection"
and "a reactive stream" — handy any time you have data already sitting in
memory (static config, a small cache) that you want to run through Reactor's
operators.
