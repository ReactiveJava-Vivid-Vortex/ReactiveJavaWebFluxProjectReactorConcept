# onErrorMap()

## In Simple Terms

`.onErrorMap(mappingFunction)` catches an error and **transforms it into a different
exception**, then re-throws that new exception downstream (i.e., the stream still
fails — just with a different, usually more meaningful, error type). It's the
reactive equivalent of catching an exception and wrapping it before re-throwing.

## Simple Example

```java
public Mono<User> getUser(String id) {
    return database.findById(id)
        .onErrorMap(SQLException.class, e ->
            new ServiceException("Failed to fetch user " + id, e)
        );
}
```

Now callers only need to know about `ServiceException`, not low-level details like
`SQLException`:

```java
getUser("123").subscribe(
    user -> System.out.println("User: " + user),
    error -> {
        if (error instanceof ServiceException) {
            System.out.println("Service-level error: " + error.getMessage());
        }
    }
);
```

## Why It Matters

`.onErrorMap()` is essential for maintaining clean **error abstraction boundaries** in
layered applications — a repository layer might throw low-level database exceptions,
but a service layer should translate those into meaningful, domain-specific
exceptions before they reach the controller/API layer.
