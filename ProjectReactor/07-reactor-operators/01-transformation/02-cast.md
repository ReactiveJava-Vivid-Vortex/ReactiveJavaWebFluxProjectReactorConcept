# cast()

## In Simple Terms

`.cast()` says "trust me, this box actually contains a more specific thing
than the label says." It's the same idea as casting in plain Java —
`(String) someObject` — just applied to items flowing through a stream. If
you're right, everything continues fine. If you're wrong, you get an error.

## Simple Example

```java
Flux<Object> objects = Flux.just("Hello", "World");

Flux<String> strings = objects.cast(String.class);

strings.subscribe(s -> System.out.println("Length: " + s.length()));
```

If you guess wrong about what's actually inside, it blows up as an error
signal instead of crashing your whole app outright — the stream just reports
the problem the reactive way:

```java
Flux<Object> mixed = Flux.just("Hello", 42); // mixing types

mixed.cast(String.class)
    .subscribe(
        s -> System.out.println(s),
        error -> System.out.println("Cast failed: " + error) // fires on the "42" item
    );
```

## Why It Matters

You'll bump into this when some API hands you a generic `Flux<Object>` (or
some other broad type) and you know, in practice, it's always going to
contain something more specific. `.cast()` lets you narrow it down cleanly
instead of writing an awkward manual cast inside a `.map()`.
