# merge()

## In Simple Terms

`Flux.merge()` starts several streams at the same time and just lets
whichever one produces something first come through first — order isn't
guaranteed, it's whoever's fastest. Think of merging lanes on a highway:
cars go through in whatever order they actually arrive, not necessarily the
order they entered.

## Simple Example

```java
Flux<String> fast = Flux.just("A1", "A2").delayElements(Duration.ofMillis(100));
Flux<String> slow = Flux.just("B1", "B2").delayElements(Duration.ofMillis(150));

Flux.merge(fast, slow)
    .subscribe(item -> System.out.println("Got: " + item));
```

Output (interleaved based on actual timing, not always the same every run):
```
Got: A1
Got: B1
Got: A2
Got: B2
```

## concat() vs merge()

| Aspect      | concat()                         | merge()                              |
|-------------|-----------------------------------|----------------------------------------|
| Subscription| Sequential (one at a time)         | Concurrent (all sources at once)      |
| Ordering    | Strictly preserved                 | Interleaved, based on timing          |
| Speed       | Slower (waits for each to finish)  | Faster (all sources work in parallel) |

## Why It Matters

`.merge()` is great when you've got several independent sources — like
calling a handful of microservices — and you don't care what order the
results show up in, you just want everything back as fast as possible.
