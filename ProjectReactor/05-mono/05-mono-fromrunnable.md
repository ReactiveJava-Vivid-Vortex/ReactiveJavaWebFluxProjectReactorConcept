# Mono.fromRunnable()

## In Simple Terms

`Mono.fromRunnable(runnable)` runs a piece of code that returns nothing, and then
completes empty — no value, just "I did the thing, and now I'm done." Use this
when you only care that something happened (logging, cleanup), not about getting
a result back.

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

A realistic use — running a side action right after a main operation finishes:

```java
saveOrder(order)
    .then(Mono.fromRunnable(() -> auditLogger.log("Order saved: " + order.getId())))
    .subscribe();
```

## Why It Matters

`Mono.fromRunnable()` is a clean way to slot plain, non-reactive "just do this"
code (like logging or metrics) into a reactive chain — especially useful with
`.then()` to make sure it runs right after something else finishes.
