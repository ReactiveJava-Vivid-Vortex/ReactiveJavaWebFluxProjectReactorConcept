# window()

## In Simple Terms

`.window(n)` is a lot like `.buffer(n)`, but instead of handing you a
finished `List`, it gives you a bunch of mini-streams ("windows"), each
holding `n` items. Each window is still a live `Flux` you can keep
processing reactively — filter it, sum it, whatever — rather than a
box already packed and sealed.

## Simple Example

```java
Flux.range(1, 9)
    .window(3)
    .flatMap(windowFlux -> windowFlux.collectList())
    .subscribe(batch -> System.out.println("Window: " + batch));
```

Output:
```
Window: [1, 2, 3]
Window: [4, 5, 6]
Window: [7, 8, 9]
```

The real difference from `.buffer()` shows up when you process each window
reactively instead of collecting it into a list first:

```java
Flux.range(1, 9)
    .window(3)
    .flatMap(windowFlux -> windowFlux.reduce(0, Integer::sum)) // sum each window
    .subscribe(sum -> System.out.println("Window sum: " + sum));
// Window sum: 6  (1+2+3)
// Window sum: 15 (4+5+6)
// Window sum: 24 (7+8+9)
```

## Why It Matters

`.window()` gives you more room to move than `.buffer()`, since each window
stays a stream instead of turning into a fully-loaded `List` right away —
that matters when windows might get huge (so you're not stuffing a giant
list into memory all at once) or when you want to run more reactive
operators (like `.reduce()` or `.filter()`) on each window separately.
