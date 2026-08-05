# Mono.fromRunnable()

## In Simple Terms

`Mono.fromRunnable(runnable)` creates a `Mono<Void>` that runs a piece of code (a
`Runnable`, which returns nothing) when subscribed, and then completes empty — no
value is emitted, just a side effect followed by successful completion. Use this when
you only care about an action happening (like logging, or triggering a cleanup), not
about getting a result back.

## Simple Example

```java
Mono<Void> mono = Mono.fromRunnable(() -> {
    System.out.println("Sending a notification email...");
});

mono.subscribe(
    v -> System.out.println("Value: " + v),           // never called (Void)
    error -> System.out.println("Error: " + error),
    () -> System.out.println("Notification task completed!")
);
```

Output:
```
Sending a notification email...
Notification task completed!
```

A realistic use case — chaining a side-effecting action after a main operation:

```java
saveOrder(order)
    .then(Mono.fromRunnable(() -> auditLogger.log("Order saved: " + order.getId())))
    .subscribe();
```

## Why It Matters

`Mono.fromRunnable()` is a clean way to plug plain, non-reactive, "fire and complete"
code (like logging or metrics) into a reactive chain, especially when combined with
`.then()` to sequence it after another operation completes.
