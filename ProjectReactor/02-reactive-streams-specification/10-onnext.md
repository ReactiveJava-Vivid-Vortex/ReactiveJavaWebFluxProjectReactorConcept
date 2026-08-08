# onNext()

## In Simple Terms

`onNext(item)` is the signal a publisher sends **every time it has a new item
ready for you.** It can fire zero, one, or many times — but never once
`onComplete()` or `onError()` has already happened.

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

For a `Mono`, this fires **at most once** (since a `Mono` is 0 or 1 value). For a
`Flux`, it can fire as many times as there are items — zero to infinity.

## Why It Matters

`onNext()` is where your actual "do something with this data" logic lives —
whether that's a `.map()`, a `.filter()`, or the final consumer inside
`.subscribe(...)`. Every operator you chain together is really just reacting to
`onNext()` signals as they flow past.
