# Project Reactor

## In Simple Terms

**Project Reactor** is a Java library that implements the Reactive Streams
specification and gives you two main building blocks: `Mono` (0 or 1 item) and `Flux`
(0 to N items). It's the reactive engine underneath Spring WebFlux, and it comes
packed with hundreds of operators to transform, combine, and control asynchronous
streams of data — without you needing to hand-write `Publisher`/`Subscriber` code
yourself.

Think of it as: **Reactive Streams defines the rules (interfaces); Project Reactor
provides a rich, production-ready toolkit built on top of those rules.**

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

Without Project Reactor, implementing correct backpressure, error handling, and
cancellation by hand (using raw `Publisher`/`Subscriber`) would be extremely tedious
and error-prone. Reactor abstracts all of that complexity behind a rich, fluent,
well-tested API — which is exactly what Spring WebFlux, R2DBC, and reactive
`WebClient` are all built on top of.
