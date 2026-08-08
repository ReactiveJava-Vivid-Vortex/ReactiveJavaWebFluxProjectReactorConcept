# Reactive Repository

## In Simple Terms

A reactive repository in Spring Data R2DBC is the reactive version of a
Spring Data JPA repository — you write an interface extending
`ReactiveCrudRepository<Entity, IdType>`, and Spring builds a fully
non-blocking implementation for you, with methods returning `Mono`/`Flux`
instead of plain objects or blocking `List`s.

## Simple Example

```java
public interface OrderRepository extends ReactiveCrudRepository<OrderEntity, String> {

    Flux<OrderEntity> findByCustomerId(String customerId);

    Mono<Long> countByStatus(String status);
}
```

Usage:

```java
@Service
public class OrderService {
    private final OrderRepository repository;

    public Flux<OrderEntity> getOrdersForCustomer(String customerId) {
        return repository.findByCustomerId(customerId); // non-blocking DB query
    }
}
```

## Why It Matters

Using a reactive repository (backed by R2DBC) instead of a traditional
blocking JPA repository keeps your entire data layer non-blocking, so
WebFlux's scalability benefits reach all the way down to the database — a
blocking JPA/Hibernate call sitting inside an otherwise reactive pipeline
would undercut that (see [[avoiding-blocking-calls]] in the Project
Reactor notes).
