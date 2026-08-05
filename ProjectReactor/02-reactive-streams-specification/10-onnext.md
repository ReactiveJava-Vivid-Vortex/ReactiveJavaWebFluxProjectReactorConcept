# onNext()

## In Simple Terms

`onNext(T item)` is the signal a `Publisher` sends to a `Subscriber` **every time a
new item is available**. It can be called zero, one, or many times during a stream's
lifetime — but never after `onComplete()` or `onError()` has been called.

```java
public interface Subscriber<T> {
    void onNext(T t); // <-- called once per item
    // ...
}
```

## Simple Example

```java
Flux.just(10, 20, 30)
    .subscribe(item -> System.out.println("onNext fired with: " + item));

// Output:
// onNext fired with: 10
// onNext fired with: 20
// onNext fired with: 30
```

For a `Mono`, `onNext()` is called **at most once** (since a `Mono` represents 0 or 1
value), whereas for a `Flux`, it can be called any number of times (0 to infinity).

## Why It Matters

`onNext()` is where your actual "do something with this piece of data" logic lives —
whether that's a `.map()` transformation, a `.filter()` check, or the final
`.subscribe(item -> ...)` consumer. Every operator you chain in a reactive pipeline
is, at its core, intercepting and reacting to `onNext()` signals as they flow
downstream.
