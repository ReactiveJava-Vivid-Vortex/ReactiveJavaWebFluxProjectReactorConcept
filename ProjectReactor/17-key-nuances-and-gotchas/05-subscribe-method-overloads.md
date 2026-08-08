# subscribe() Method Overloads

## In Simple Terms

`.subscribe()` comes in several flavors, and which one you pick determines
how much control you get — and how easy it is to accidentally let an error
slip by unnoticed. From least to most control:

```java
mono.subscribe();
// Fire-and-forget. If it errors, the exception is logged (not silently lost),
// but you have NO way to react to it in your own code.

mono.subscribe(value -> handle(value));
// Handles the value. WARNING: if this Mono errors, the error still just gets
// logged by Reactor's default handler — your code never sees it either.

mono.subscribe(
    value -> handle(value),
    error -> handleError(error)
);
// NOW you're actually handling both outcomes — this is the safe minimum
// for any production code.

mono.subscribe(
    value -> handle(value),
    error -> handleError(error),
    () -> System.out.println("Done!")
);
// Adds an explicit completion callback too — useful when "it finished
// successfully" itself needs an action (e.g., closing a resource).
```

## Simple Example

```java
Mono<Integer> risky = Mono.fromCallable(() -> 10 / 0); // will error

// BAD: error is swallowed by your code (Reactor still logs it internally,
// but your application logic never reacts to it)
risky.subscribe(value -> System.out.println("Value: " + value));

// GOOD: your code explicitly reacts to the failure
risky.subscribe(
    value -> System.out.println("Value: " + value),
    error -> System.out.println("Handled failure: " + error.getMessage())
);
```

## Why It Matters

In a Spring WebFlux app, you rarely call `.subscribe()` yourself — the
framework does it for you when handling a request, and its own error
handling (`@ControllerAdvice`, default error responses) takes it from
there. But in standalone code, scheduled tasks, or event listeners where
*you* call `.subscribe()` directly, always give it at least an error
handler — otherwise failures just get quietly logged by Reactor internally
instead of being handled by your own logic, which can make bugs very easy
to miss.
