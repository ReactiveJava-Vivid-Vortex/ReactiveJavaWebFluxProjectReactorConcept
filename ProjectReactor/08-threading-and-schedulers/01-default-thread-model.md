# Default Thread Model

## In Simple Terms

By default, a Reactor pipeline just stays on whatever thread called
`.subscribe()` — nothing hops to another thread automatically. That trips
up a lot of newcomers, since "reactive" sounds like it should mean
"multi-threaded" or "parallel." It doesn't, not unless you explicitly ask
for it.

## Simple Example

```java
public class DefaultThreadDemo {
    public static void main(String[] args) {
        System.out.println("Main thread: " + Thread.currentThread().getName());

        Mono.just("Hello")
            .map(value -> {
                System.out.println("map() runs on: " + Thread.currentThread().getName());
                return value.toUpperCase();
            })
            .subscribe(value -> {
                System.out.println("subscribe() runs on: " + Thread.currentThread().getName());
            });
    }
}
```

Output:
```
Main thread: main
map() runs on: main
subscribe() runs on: main
```

Everything ran right there on the `main` thread — no automatic switching
happened.

## Why It Matters

This default is important to understand: reactive programming's real power
comes from **not tying up threads while waiting**, not from magically
spreading work across many threads. If you want work to run somewhere else
— say, so a heavy task doesn't clog up an important thread — you have to
say so explicitly with `subscribeOn()` or `publishOn()`.
