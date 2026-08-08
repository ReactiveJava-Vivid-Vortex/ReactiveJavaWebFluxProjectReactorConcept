# Project Reactor

## In Simple Terms

**Project Reactor** is a Java library that gives you two main tools —
`Mono` (0 or 1 item) and `Flux` (0 to many items) — plus hundreds of ready-made
operators for shaping, combining, and controlling streams of data. It's the
engine that Spring WebFlux runs on, and it saves you from ever having to
hand-write your own `Publisher`/`Subscriber` code.

Simple way to think about it: **Reactive Streams is the rulebook. Project Reactor
is the fully-built toolkit that follows those rules for you.**

## Simple Example

```java
import reactor.core.publisher.Flux;

public class ReactorDemo {
    public static void main(String[] args) {
        Flux.just("Reactor", "makes", "reactive", "programming", "easy")
            .map(String::toUpperCase)
            .filter(word -> word.length() > 5)
            .subscribe(System.out::println);
    }
}
```

Output:
```
REACTOR
REACTIVE
PROGRAMMING
```

## Why It Matters

Without Reactor, you'd have to build correct backpressure, error handling, and
cancellation entirely by hand — tedious and easy to get wrong. Reactor hides all
of that behind a clean, well-tested API, which is exactly what Spring WebFlux,
R2DBC, and `WebClient` are all built on top of.
