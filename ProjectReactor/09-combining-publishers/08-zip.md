# zip()

## In Simple Terms

`Flux.zip()` pairs items up from multiple streams like a zipper closing —
item 1 from each source pairs together, item 2 from each pairs together,
and so on. It waits until *all* sources have their next item ready before
producing a pair, and it stops as soon as the shortest source runs out.

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

If one source runs out of items sooner, `zip()` just stops there:

```java
Flux<String> names = Flux.just("Alice", "Bob");     // only 2 items
Flux<Integer> ages = Flux.just(30, 25, 35);          // 3 items

Flux.zip(names, ages).subscribe(t -> System.out.println(t));
// Only 2 pairs emitted — "Charlie"/35 is dropped, since names ran out
```

You can also give it a function to combine the pair into something nicer
than a raw tuple:

```java
Flux.zip(names, ages, (name, age) -> name + " (" + age + ")")
    .subscribe(System.out::println);
```

## Why It Matters

`.zip()` is the standard way to line up results from separate, independent
calls that ran at the same time — like fetching a user's profile and their
order history in parallel, then combining both once they're both ready.
