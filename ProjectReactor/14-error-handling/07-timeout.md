# timeout()

## In Simple Terms

`.timeout()` makes a `Mono`/`Flux` give up and fail if it doesn't produce
anything within a set amount of time — like hanging up a phone call if
nobody answers after a while. It's essential for stopping things from
waiting forever on a service that just never responds.

## Simple Example

```java
Mono<String> slowCall = Mono.delay(Duration.ofSeconds(5)).map(t -> "Finally done");

slowCall
    .timeout(Duration.ofSeconds(2)) // fail if it takes longer than 2 seconds
    .subscribe(
        result -> System.out.println("Result: " + result),
        error -> System.out.println("Timed out: " + error.getMessage())
    );
```

Output (after 2 seconds, not 5):
```
Timed out: Did not observe any item or terminal signal within 2000ms
```

Combining `.timeout()` with a fallback:

```java
slowCall
    .timeout(Duration.ofSeconds(2))
    .onErrorResume(TimeoutException.class, e -> Mono.just("Fallback: took too long"))
    .subscribe(System.out::println);
```

## Why It Matters

`.timeout()` is a must-have safety net for any call to an outside system —
without it, one unresponsive dependency could leave requests hanging
forever, eventually eating up resources (connections, threads) across your
whole app.
