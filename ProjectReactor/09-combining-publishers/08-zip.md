# zip()

## In Simple Terms

`Flux.zip(source1, source2, ...)` combines multiple publishers **pairwise** — it
waits until *all* sources have produced their Nth item, then emits them together as a
tuple. It stops as soon as the **shortest** source completes.

## Simple Example

```java
Flux<String> names = Flux.just("Alice", "Bob", "Charlie");
Flux<Integer> ages = Flux.just(30, 25, 35);

Flux.zip(names, ages)
    .subscribe(tuple -> System.out.println(tuple.getT1() + " is " + tuple.getT2() + " years old"));
```

Output:
```
Alice is 30 years old
Bob is 25 years old
Charlie is 35 years old
```

If one source has fewer items, `zip()` stops there:

```java
Flux<String> names = Flux.just("Alice", "Bob");     // only 2 items
Flux<Integer> ages = Flux.just(30, 25, 35);          // 3 items

Flux.zip(names, ages).subscribe(t -> System.out.println(t));
// Only 2 pairs emitted — "Charlie"/35 is dropped, since names ran out
```

Using a combiner function instead of a raw tuple:

```java
Flux.zip(names, ages, (name, age) -> name + " (" + age + ")")
    .subscribe(System.out::println);
```

## Why It Matters

`.zip()` is the standard way to **pair up results from independent, parallel calls**
— e.g., fetching a user's profile and their order history concurrently, then
combining both into a single combined response once both are available.
