# checkpoint()

## In Simple Terms

`.checkpoint()` drops a labeled signpost into your pipeline so that, if
something goes wrong later, Reactor can point back and say "the problem is
near this signpost." Normal stack traces from reactive code tend to be a
mess of internal framework noise — `.checkpoint()` gives you something
readable to look at instead.

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

When the divide-by-zero error happens, the stack trace now mentions
`"division-step"` — making it much easier to spot exactly which part of a
long chain caused the failure, instead of trying to decode generic internal
Reactor class names.

For even more detail (at some cost to performance), use
`.checkpoint("desc", true)` to capture the exact code location, or turn on
`Hooks.onOperatorDebug()` globally during development to get full traces
automatically (best avoided in production — it's expensive).

## Why It Matters

Reactive pipelines can be genuinely hard to debug, since errors often show
up far away from where they actually started, and the default stack trace
is cluttered with Reactor's internal plumbing. `.checkpoint()` is a small,
targeted way to make failures in long or tangled pipelines much easier to
trace back to their real source.
