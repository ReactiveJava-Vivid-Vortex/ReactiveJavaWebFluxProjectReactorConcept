# checkpoint()

## In Simple Terms

`.checkpoint()` (optionally with a description string) adds a marker into a
pipeline's stack trace so that, if an error occurs, Reactor can tell you which part
of your reactive chain it came from. Normal Java stack traces for reactive code are
often unhelpful (full of internal Reactor framework frames); `.checkpoint()` inserts
a readable, meaningful reference point.

## Simple Example

```java
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .checkpoint("division-step")
    .subscribe(
        result -> System.out.println("Result: " + result),
        error -> error.printStackTrace()
    );
```

When the `ArithmeticException` occurs, the resulting stack trace includes a line
referencing `"division-step"`, making it much easier to pinpoint exactly which
operator in a long chain caused the failure — instead of guessing from generic
internal Reactor class names.

For even more detail (at a performance cost), use `.checkpoint("desc", true)` to
capture the actual code location, or globally enable
`Hooks.onOperatorDebug()` during development to get full assembly traces
automatically (not recommended for production due to overhead).

## Why It Matters

Debugging reactive pipelines can be notoriously hard because errors often surface far
from where they originated, and default stack traces are cluttered with internal
Reactor machinery. `.checkpoint()` is a lightweight, targeted way to make failures in
long or complex pipelines much easier to trace back to their source.
