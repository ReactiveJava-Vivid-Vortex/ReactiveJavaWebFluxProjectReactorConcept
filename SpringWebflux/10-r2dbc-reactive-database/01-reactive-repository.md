# Reactive Repository

## In Simple Terms

A **reactive repository** in Spring Data R2DBC is the reactive equivalent of a
Spring Data JPA repository — you define an interface extending
`ReactiveCrudRepository<Entity, IdType>`, and Spring generates a fully non-blocking
implementation, with methods returning `Mono`/`Flux` instead of plain objects or
blocking `List`s.

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

Using a reactive repository (backed by R2DBC) instead of a traditional blocking JPA
repository keeps your entire data access layer non-blocking, preserving the
scalability benefits of WebFlux all the way down to the database — a blocking
JPA/Hibernate repository call inside an otherwise reactive pipeline would undermine
that benefit (see [[avoiding-blocking-calls]] in the ProjectReactor notes).
