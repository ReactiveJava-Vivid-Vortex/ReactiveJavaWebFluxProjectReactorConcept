# doOnError()

## In Simple Terms

`.doOnError(consumer)` runs a side effect (typically logging) when an error passes
through this point in the pipeline — **without** handling or recovering from the
error. The error still propagates downstream afterward, unlike `onErrorResume()`
which actually replaces it.

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

Notice the error is still delivered to the final `subscribe()` error handler — 
`.doOnError()` only observes it, it doesn't consume or replace it.

## Why It Matters

`.doOnError()` is the standard place to add centralized error **logging** (e.g., to a
monitoring system) at any point in a pipeline, while leaving the actual error handling
/recovery decision (via `onErrorResume`, `onErrorReturn`, etc.) to a separate,
dedicated operator further downstream.
