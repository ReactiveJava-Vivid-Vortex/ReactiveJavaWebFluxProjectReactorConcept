# Reactive Pipeline

## In Simple Terms

A **reactive pipeline** is just a chain of operators attached to a `Mono` or
`Flux` — a description of what should happen to the data, **once someone
subscribes.** Think of it like setting up stations on a factory assembly line:
you build all the stations first (filter, map, and so on), but nothing actually
moves until you switch the conveyor belt on (`.subscribe()`).

## Simple Example

```java
Flux<String> pipeline = Flux.just("apple", "banana", "cherry", "date")
    .filter(fruit -> fruit.length() > 4)   // station 1: filter
    .map(String::toUpperCase)              // station 2: transform
    .doOnNext(f -> System.out.println("About to emit: " + f)); // station 3: side effect

// NOTHING has happened yet! The pipeline is just a description.

pipeline.subscribe(fruit -> System.out.println("Received: " + fruit));

// NOW the data actually flows through every station.
```

Output:
```
About to emit: APPLE
Received: APPLE
About to emit: BANANA
Received: BANANA
About to emit: CHERRY
Received: CHERRY
About to emit: DATE
Received: DATE
```

Notice each item goes through the *whole* pipeline (filter → map → doOnNext →
subscribe) one at a time — it's not "filter everything, then map everything."

## Why It Matters

Once you get that a pipeline is just a blueprint until someone subscribes, a lot
of confusing beginner moments make sense — like "why is my pipeline doing
nothing?" (usually: nobody called `.subscribe()`).
