# onComplete()

## In Simple Terms

`onComplete()` is the signal a publisher sends to say **"I'm done — nothing more
is coming, and it all worked."** It only ever fires once, and only if nothing
went wrong along the way.

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

Worth noting: `Mono.empty()` still fires `onComplete()` even though it never sent
a single item — "finished with nothing to give you" still counts as a success.

## Why It Matters

`onComplete()` is your cue to run any cleanup that should only happen when things
went right — different from `doFinally()`, which runs no matter what (success,
error, or cancellation). It's often used to know when it's safe to close a
resource or mark a job as finished.
