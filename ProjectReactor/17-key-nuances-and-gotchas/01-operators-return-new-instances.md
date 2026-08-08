# Operators Return New Instances (Immutability)

## In Simple Terms

A `Mono`/`Flux` never changes once it's built. Calling something like
`.map()` or `.filter()` doesn't modify the original — it hands you back a
*brand-new* `Mono`/`Flux` that wraps the old one. If you don't grab that
new result (or chain straight onto it), your transformation just... doesn't
happen. Nothing warns you it got skipped.

This is, by far, the most common "why isn't my reactive code doing
anything?!" trap for people coming from regular imperative Java.

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

In practice, you sidestep this bug naturally just by chaining everything
together in one fluent line:

```java
Flux.just(1, 2, 3)
    .map(n -> n * 100)   // each operator returns a new Flux, chained immediately
    .filter(n -> n > 150)
    .subscribe(n -> System.out.println("Got: " + n));
```

## Why It Matters

Because Reactor types never change once built, a `Mono`/`Flux` is
completely safe to store, pass around, and reuse — nobody can accidentally
mess it up by calling an operator on it. The catch is this specific
beginner trap: **an operator call you don't assign or chain is a silent
no-op** — not a compile error, not a runtime exception, just... nothing.
Always chain, or always reassign.
