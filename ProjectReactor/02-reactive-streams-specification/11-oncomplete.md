# onComplete()

## In Simple Terms

`onComplete()` is the signal a `Publisher` sends to say **"I'm done — there is no
more data coming, and everything succeeded."** It is called at most once, and only if
the stream didn't fail with an error.

```java
public interface Subscriber<T> {
    void onComplete(); // <-- terminal success signal
}
```

## Simple Example

```java
Flux.just("a", "b", "c")
    .subscribe(
        item -> System.out.println("Item: " + item),
        error -> System.out.println("Error: " + error),
        () -> System.out.println("All done, successfully!") // onComplete callback
    );

// Output:
// Item: a
// Item: b
// Item: c
// All done, successfully!
```

Note: a `Mono.empty()` still calls `onComplete()` even though it never emits any item
— "completed with zero items" is a perfectly valid, successful outcome.

## Why It Matters

`onComplete()` is your cue to run any "finalization" logic that should only happen on
**success** — as opposed to `doFinally()`, which runs on success, error, *or*
cancellation. It's commonly used to know when it's safe to, say, close a resource or
mark a batch job as finished.
