# Common Misconceptions About Reactive Programming

## In Simple Terms

Reactive programming gets misunderstood a lot. Here are the myths people
believe most often, and what's actually true.

### Myth 1: "Reactive is always faster"

**Truth:** No — a single reactive database call isn't any quicker than a normal
one. The network and database are just as slow either way. What reactive
actually improves is how many requests you can handle **at the same time**,
because threads aren't wasted sitting around waiting.

### Myth 2: "Reactive code automatically runs on multiple threads"

**Truth:** No — by default, a reactive pipeline runs on whichever thread called
`subscribe()`, one step at a time. Nothing runs in parallel unless you
deliberately ask for it with `subscribeOn()` or `parallel()`.

```java
Mono.just("hello")
    .map(String::toUpperCase) // runs on the SAME thread that subscribed, by default
    .subscribe(System.out::println);
```

### Myth 3: "One blocking call won't hurt, it's just one line"

**Truth:** It can hurt a lot. A single blocking database call inside a reactive
pipeline can freeze one of your few precious threads — and that same thread might
be handling many *other* unrelated requests too. This is one of the sneakiest,
most damaging mistakes people make with reactive code.

### Myth 4: "Reactive programming fixes bad design"

**Truth:** No — if your database is slow or another service is unreliable,
reactive code still has to wait for it. It just doesn't waste a thread while
waiting.

### Myth 5: "You have to make everything reactive to benefit"

**Truth:** No — you don't have to convert your whole app overnight. Reactive
pays off most where you have lots of traffic and lots of waiting; other parts of
your system can stay exactly as they are if that's simpler.

## Why It Matters

Believing these myths leads to two common traps: expecting reactive code to be a
free speed boost, or accidentally blocking inside a reactive pipeline and then
wondering why the whole app seems to freeze under load. Once you understand what
reactive actually does — non-blocking scheduling, not magic speed or automatic
parallelism — you avoid both traps.
