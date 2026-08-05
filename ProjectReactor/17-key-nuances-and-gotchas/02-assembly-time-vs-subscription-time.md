# Assembly Time vs Subscription Time

## In Simple Terms

Every reactive pipeline has two completely separate "times" you need to keep apart
in your head:

- **Assembly time** — when the pipeline is *built* (`Flux.just(...).map(...)...`).
  This runs exactly once, top-to-bottom, like normal Java code, the instant that
  line executes.
- **Subscription time** — when the pipeline actually *runs*, once `.subscribe()` is
  called. This is when your lambdas' *bodies* actually execute, potentially many
  times (once per item, or once per subscriber).

Confusing these two is the deeper reason behind laziness surprises — code inside a
lambda passed to an operator doesn't run at assembly time, only later, at
subscription time.

## Simple Example

```java
System.out.println("1. Assembly starts");

Flux<String> pipeline = Flux.just("a", "b")
    .map(s -> {
        System.out.println("3. Lambda body runs (subscription time!): " + s);
        return s.toUpperCase();
    });

System.out.println("2. Assembly finished — pipeline built, nothing executed yet");

pipeline.subscribe(s -> System.out.println("4. Received: " + s));
```

Output:
```
1. Assembly starts
2. Assembly finished — pipeline built, nothing executed yet
3. Lambda body runs (subscription time!): a
4. Received: A
3. Lambda body runs (subscription time!): b
4. Received: B
```

Notice line "2" prints **before** any lambda body runs — assembly only wires the
pipeline together; it doesn't execute your logic.

## Why It Matters

This distinction explains a whole category of confusing behavior:

- Code **outside** any operator lambda (like a `System.out.println` between two
  chained calls at the top level of your method) runs at **assembly time** — once,
  immediately.
- Code **inside** an operator lambda (`.map(x -> ...)`, `.filter(x -> ...)`,
  `.flatMap(x -> ...)`) runs at **subscription time** — later, per item, possibly
  never if nobody subscribes, and possibly many times if multiple subscribers
  attach to a cold publisher.

Whenever a reactive pipeline "does something at the wrong time," ask: **"is this
code at assembly time, or subscription time?"** — it's very often the answer.
