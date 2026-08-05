# Default Thread Model

## In Simple Terms

By default, a Project Reactor pipeline runs **entirely on the thread that called
`.subscribe()`** — no automatic thread switching happens unless you explicitly ask
for it (via `subscribeOn()`/`publishOn()`). This surprises many beginners who assume
"reactive" automatically means "multi-threaded" or "parallel."

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

Everything runs on the `main` thread — nothing switched threads automatically.

## Why It Matters

Understanding this default is crucial: reactive programming's power comes from
**not blocking** threads, not from automatically parallelizing work. If you need work
to run on a different thread (e.g., to avoid blocking an event-loop thread with a
CPU-heavy task), you must explicitly say so using `subscribeOn()` or `publishOn()`.
