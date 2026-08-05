# The Three Signal Types — The Whole Vocabulary of Reactive Streams

## In Simple Terms

Here is the single most useful fact to memorize in this entire course: **no matter
how complicated a reactive pipeline looks — 20 chained operators, nested `flatMap`s,
combined sources — everything that ever happens inside it is built from only three
kinds of data signals:**

1. **`onNext(item)`** — "here's a new item" — can happen **zero or more times**.
2. **`onComplete()`** — "all done, it worked" — a **terminal** signal, happens **at
   most once**.
3. **`onError(throwable)`** — "something went wrong" — also **terminal**, happens
   **at most once**.

That's it. There is no fourth "data" signal. Every operator you will ever learn
(`map`, `filter`, `flatMap`, `retry`, `buffer`...) is, underneath, just a piece of
code that watches these three signals go by and decides what to do with each one.

There is technically one more method on a `Subscriber` — `onSubscribe(subscription)`
— but it's not a *data* signal. It's the one-time "handshake" that happens before
any data flows, handing the subscriber its `Subscription` (its remote control for
`request(n)`/`cancel()`). Once that handshake is done, everything else that happens
is one of the three signals above.

## The Formal Rule (Reactive Streams Grammar)

The Reactive Streams specification writes this as one strict, memorable grammar:

```
onSubscribe onNext* (onError | onComplete)?
```

Read it like a sentence: "First `onSubscribe`, then zero-or-more `onNext`s, then
*optionally* — but at most once — either an `onError` or an `onComplete`."

Three hard rules fall directly out of this grammar, and they explain a huge amount
of reactive behavior:

- **`onNext` can repeat, but the terminal signals can't.** You can get 0, 1, or a
  million `onNext` calls, but `onComplete`/`onError` each happen at most once.
- **`onError` and `onComplete` are mutually exclusive.** A stream ends in success
  *or* failure — never both, and there's no "partial success with a warning."
- **Nothing is ever signaled after a terminal signal.** Once `onComplete()` or
  `onError()` fires, the stream is over — no more `onNext`, no second terminal
  signal, nothing. (If you never see either, e.g. an infinite `Flux.interval()`,
  the stream simply hasn't ended yet.)

## Simple Example

```java
Flux.just(1, 2, 3)
    .subscribe(
        value  -> System.out.println("onNext: " + value),   // called 3 times
        error  -> System.out.println("onError: " + error),  // NOT called (no failure)
        ()     -> System.out.println("onComplete!")          // called exactly once, at the end
    );

// Output:
// onNext: 1
// onNext: 2
// onNext: 3
// onComplete!
```

Now the failure case — notice `onNext` still may fire a few times *before* the
terminal signal, but `onComplete` never fires once `onError` has:

```java
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n) // throws ArithmeticException when n == 0
    .subscribe(
        value -> System.out.println("onNext: " + value),
        error -> System.out.println("onError: " + error.getMessage())
    );

// Output:
// onNext: 10
// onNext: 5
// onError: / by zero
// (the "4" is NEVER processed — onError ended the stream first)
```

## Why This One Rule Explains So Much

Once this clicks, a lot of previously-confusing reactive behavior suddenly makes
sense:

- **`Mono<T>` is just this same grammar with a stricter cap on `onNext`** — at most
  **one** `onNext`, so a `Mono` has exactly three possible outcomes: value+complete,
  empty (complete only, no `onNext`), or error. See [[mono-lifecycle]].
- **`.doOnNext()`, `.doOnComplete()`, `.doOnError()`, `.doFinally()`** are simply
  hooks that intercept one (or, for `doFinally`, any) of these exact signals — see
  the Reactor Operators topic's side-effect operators.
- **`StepVerifier`** in the Testing topic is, at its core, asserting that this exact
  grammar played out the way you expected — `.expectNext(...)` checks `onNext`
  calls, `.verifyComplete()`/`.expectError()` checks which terminal signal fired.
- **Why you can't "recover and keep going" from an error by default** — once
  `onError` fires, the grammar says the stream is over. Operators like
  `onErrorResume()` don't reopen the *same* stream; they *switch to a brand new one*
  that starts its own fresh `onSubscribe → onNext* → terminal` sequence.

## Why It Matters

This isn't just trivia — it's the shortest possible mental model for the entire
Reactive Streams ecosystem. Whenever you're confused about what an operator does,
ask: **"which of the three signals is it reacting to, and is it changing the
data, or just observing it?"** That question, applied consistently, will get you
through 90% of reactive debugging.
