# onErrorMap()

## In Simple Terms

`.onErrorMap()` catches an error and swaps it for a *different* error
before sending it downstream — the stream still fails, just with something
more meaningful attached. It's the same idea as catching an exception,
wrapping it in a clearer one, and throwing that instead.

## Simple Example

```java
public Mono<User> getUser(String id) {
    return database.findById(id)
        .onErrorMap(SQLException.class, e ->
            new ServiceException("Failed to fetch user " + id, e)
        );
}
```

Now callers only need to know about `ServiceException`, not low-level
details like `SQLException`:

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

`.onErrorMap()` keeps things clean across layers of an app — a repository
might throw raw database exceptions, but a service layer should translate
those into meaningful, domain-specific errors before they ever reach the
controller or API layer.
