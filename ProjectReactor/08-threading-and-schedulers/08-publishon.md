# publishOn()

## In Simple Terms

`.publishOn()` switches the thread for everything *after* it in the
pipeline — like handing off a relay baton at a specific checkpoint. Only
the code downstream of this point feels the switch; everything before it
keeps running exactly where it was.

## Simple Example

```java
Flux.range(1, 3)
    .doOnNext(n -> System.out.println("Before publishOn: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())
    .doOnNext(n -> System.out.println("After publishOn: " + Thread.currentThread().getName()))
    .subscribe();
```

Output:
```
Before publishOn: main
After publishOn: boundedElastic-1
Before publishOn: main
After publishOn: boundedElastic-1
Before publishOn: main
After publishOn: boundedElastic-1
```

You can use `.publishOn()` more than once in the same chain, to switch
threads multiple times — say, do CPU work on `parallel()`, then hop over to
`boundedElastic()` for a blocking database write.

## Why It Matters

`.publishOn()` gives you precise control over exactly where in the chain a
thread switch happens — useful when only one piece of your pipeline
actually needs a special kind of thread (say, only the final database write
needs `boundedElastic()`, while the earlier steps are fine staying put).
