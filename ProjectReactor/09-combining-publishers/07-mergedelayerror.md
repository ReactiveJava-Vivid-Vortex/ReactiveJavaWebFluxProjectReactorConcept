# mergeDelayError()

## In Simple Terms

`Flux.mergeDelayError()` behaves like `merge()`, except if one of the
sources fails, the failure gets held back until every other source has had
a chance to finish, one way or another — so a problem in one source doesn't
cut off results that are still coming in fine from the rest.

## Simple Example

```java
Flux<String> ok = Flux.just("A", "B", "C").delayElements(Duration.ofMillis(50));
Flux<String> failing = Flux.<String>error(new RuntimeException("Service down"))
    .delaySubscription(Duration.ofMillis(75));

Flux.mergeDelayError(2, ok, failing)
    .subscribe(
        item -> System.out.println("Got: " + item),
        error -> System.out.println("Error at the end: " + error.getMessage())
    );
```

Output:
```
Got: A
Got: B
Got: C
Error at the end: Service down
```

With plain `.merge()`, the failure from `failing` would immediately shut
down the whole stream — probably before `ok` even had a chance to emit all
its items.

## Why It Matters

`.mergeDelayError()` matters when you're combining results from several
independent, unreliable sources — like pulling data from a handful of
microservices — where one service having a bad day shouldn't stop you from
still getting the good results from everyone else. You only find out about
the failure after everything else finished.
