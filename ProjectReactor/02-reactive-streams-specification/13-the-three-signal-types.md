# The Three Signal Types — The Whole Vocabulary of Reactive Streams

## In Simple Terms

Here's one fact worth memorizing above everything else in this course: **no
matter how complicated a reactive pipeline looks — 20 chained operators, nested
`flatMap`s, five combined sources — everything happening inside it boils down to
just three kinds of signals:**

1. **`onNext(item)`** — "here's a new item" — can happen zero or more times.
2. **`onComplete()`** — "all done, it worked" — happens at most once.
3. **`onError(throwable)`** — "something went wrong" — also happens at most once.

That's the whole list. Every operator you'll ever learn (`map`, `filter`,
`flatMap`, `retry`, `buffer`...) is really just a small piece of code watching
these three signals go by and deciding what to do with each one.

There's technically one more method — `onSubscribe(subscription)` — but it's not
a *data* signal. It's the one-time handshake that happens before anything else,
handing the subscriber its remote control. After that handshake, everything else
is one of the three signals above.

## The Formal Rule

Reactive Streams writes this down as one short, strict rule:

```
onSubscribe onNext* (onError | onComplete)?
```

In plain words: "first `onSubscribe`, then zero or more `onNext`s, then —
optionally, and at most once — either an `onError` or an `onComplete`."

Three things fall straight out of this rule:

- **`onNext` can repeat, the other two can't.** You might get 0, 1, or a million
  `onNext` calls, but `onComplete`/`onError` each happen at most once.
- **`onError` and `onComplete` never both happen.** A stream ends in success or
  in failure — never a mix of both.
- **Nothing happens after the ending.** Once `onComplete()` or `onError()` fires,
  that's it — no more items, no second ending signal. (If you never see either
  one, like with an endless `Flux.interval()`, the stream just hasn't ended yet.)

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

Now the failure case — notice `onNext` can still fire a few times *before* the
ending, but `onComplete` never fires once `onError` already has:

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
// (the "4" is never processed — onError already ended the stream)
```

## Why This One Rule Explains So Much

Once this sinks in, a lot of previously-confusing behavior suddenly makes sense:

- **A `Mono<T>` is just this same rule with a tighter limit on `onNext`** — at
  most one, instead of unlimited. So a `Mono` only ever has three outcomes: value,
  empty (no value, just a success), or error. See [[mono-lifecycle]].
- **`.doOnNext()`, `.doOnComplete()`, `.doOnError()`, `.doFinally()`** are just
  little hooks that watch for one (or, for `doFinally`, any) of these exact
  signals — see the side-effect operators in the Reactor Operators topic.
- **`StepVerifier`** in the Testing topic is really just checking that this exact
  rule played out the way you expected — `.expectNext(...)` checks `onNext`
  calls, `.verifyComplete()`/`.expectError()` checks which ending happened.
- **Why you can't "recover and keep going" from an error by default** — once
  `onError` fires, the stream is over, full stop. Operators like
  `onErrorResume()` don't magically un-end that stream; they switch to a
  brand-new one that starts its own fresh sequence.

## Why It Matters

This isn't trivia — it's the shortest, simplest way to understand all of Reactive
Streams. Any time you're confused about what an operator is doing, just ask:
**"which of the three signals is it reacting to, and is it changing the data or
just watching it?"** That one question will answer most of your questions.
