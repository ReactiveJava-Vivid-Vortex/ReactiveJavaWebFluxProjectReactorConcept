# doOnError()

## In Simple Terms

`.doOnError()` lets you react to an error going by — usually to log it —
without actually catching or fixing it. The error keeps traveling downstream
afterward exactly as before. This is different from `.onErrorResume()`,
which actually replaces the error with something else.

## Simple Example

```java
Mono.error(new RuntimeException("Database connection failed"))
    .doOnError(error -> System.out.println("Logging error: " + error.getMessage()))
    .subscribe(
        value -> System.out.println("Value: " + value),
        error -> System.out.println("Final handler saw: " + error.getMessage())
    );
```

Output:
```
Logging error: Database connection failed
Final handler saw: Database connection failed
```

Notice the error still reaches the final `subscribe()` error handler —
`.doOnError()` only watches it go by, it doesn't swallow or change it.

## Why It Matters

`.doOnError()` is the natural place to add central error **logging** (say,
to a monitoring system), while leaving the actual decision of how to
recover — via `onErrorResume`, `onErrorReturn`, and friends — to a separate
operator further down the chain.
