# cast()

## In Simple Terms

`.cast(Class<E>)` changes the **declared type** of a `Mono`/`Flux` from one type to
another, at runtime, similar to a Java type cast (`(SomeType) obj`). It's useful when
you know (or expect) that the emitted items are actually instances of a more specific
subtype than what's currently declared.

## Simple Example

```java
Flux<Object> objects = Flux.just("Hello", "World");

Flux<String> strings = objects.cast(String.class);

strings.subscribe(s -> System.out.println("Length: " + s.length()));
```

If the actual runtime type doesn't match, you'll get a `ClassCastException` delivered
as an `onError()` signal, just like a normal cast would throw:

```java
Flux<Object> mixed = Flux.just("Hello", 42); // mixing types

mixed.cast(String.class)
    .subscribe(
        s -> System.out.println(s),
        error -> System.out.println("Cast failed: " + error) // fires on the "42" item
    );
```

## Why It Matters

`.cast()` is handy when working with generic APIs that return `Flux<Object>` or a
common supertype, and you need to narrow the type for further type-safe processing
downstream — without writing a manual `.map(o -> (String) o)`.
