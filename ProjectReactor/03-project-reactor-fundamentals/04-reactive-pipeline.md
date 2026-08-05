# Reactive Pipeline

## In Simple Terms

A **reactive pipeline** is a chain of operators applied to a `Mono` or `Flux`,
describing a series of transformations data will go through — **once someone
subscribes**. It's like assembling a factory assembly line: you set up all the
stations first (map, filter, etc.), but nothing moves down the line until the
conveyor belt is switched on (`.subscribe()`).

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

Notice each item flows through the *entire* pipeline (filter → map → doOnNext →
subscribe) one at a time, rather than all items being filtered first, then all mapped.

## Why It Matters

Understanding that a pipeline is just a **blueprint** until subscribed to (see "Lazy
Execution") explains many surprising behaviors beginners run into — like a pipeline
seemingly "doing nothing" because they forgot to call `.subscribe()`.
