# Assembly Time vs Subscription Time

## In Simple Terms

Every reactive pipeline has two completely separate moments you need to
keep straight in your head:

- **Assembly time** — when the pipeline is *built* (`Flux.just(...).map(...)...`).
  This happens exactly once, top to bottom, the instant that line of code
  runs, just like regular Java.
- **Subscription time** — when the pipeline actually *runs*, once
  `.subscribe()` gets called. This is when the bodies of your lambdas
  actually execute — possibly many times, once per item, or once per
  subscriber.

Mixing these two up is the real reason behind most "laziness surprises" —
code inside a lambda you hand to an operator doesn't run when you build the
pipeline, only later, when someone actually subscribes.

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

Notice line "2" prints *before* any lambda body runs at all — assembling
the pipeline just wires it together, it doesn't run any of your logic yet.

## Why It Matters

This distinction explains a whole family of confusing behavior:

- Code **outside** any operator lambda (like a stray `System.out.println`
  sitting between two chained calls) runs at assembly time — once,
  immediately.
- Code **inside** an operator lambda (`.map(x -> ...)`, `.filter(x -> ...)`,
  `.flatMap(x -> ...)`) runs at subscription time — later, per item,
  possibly never if nobody ever subscribes, and possibly many times if
  several subscribers attach to a cold publisher.

Whenever a reactive pipeline seems to "do something at the wrong time,"
ask yourself: is this assembly time, or subscription time? Nine times out
of ten, that's your answer.
