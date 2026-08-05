# Operators Return New Instances (Immutability)

## In Simple Terms

A `Mono`/`Flux` is **immutable**. Calling an operator like `.map()` or `.filter()`
does **not** modify the original object — it returns a **brand-new**
`Mono`/`Flux` wrapping your original one. If you don't capture (or chain onto) that
return value, your transformation simply never happens, and nothing tells you it
was ignored.

This is, by a wide margin, the most common "why isn't my reactive code doing
anything?!" bug for beginners coming from imperative Java.

## Simple Example

```java
Flux<Integer> numbers = Flux.just(1, 2, 3);

// BUG: .map() returns a NEW Flux — this line's result is thrown away!
numbers.map(n -> n * 100);

numbers.subscribe(n -> System.out.println("Got: " + n));
// Output: Got: 1, Got: 2, Got: 3   ← the *100 never happened!
```

```java
Flux<Integer> numbers = Flux.just(1, 2, 3);

// FIX: capture (or chain onto) the returned Flux
Flux<Integer> scaled = numbers.map(n -> n * 100);

scaled.subscribe(n -> System.out.println("Got: " + n));
// Output: Got: 100, Got: 200, Got: 300   ← correct!
```

In practice you almost always avoid this bug naturally by chaining everything in
one fluent expression:

```java
Flux.just(1, 2, 3)
    .map(n -> n * 100)   // each operator returns a new Flux, chained immediately
    .filter(n -> n > 150)
    .subscribe(n -> System.out.println("Got: " + n));
```

## Why It Matters

Because Reactor types are immutable, a `Mono`/`Flux` is completely safe to store,
pass around, and reuse — nobody can accidentally corrupt it by calling an operator
on it. The tradeoff is this specific beginner trap: **an operator call that isn't
assigned or chained is a silent no-op**, not a compile error and not a runtime
exception. Always chain, or always reassign.
