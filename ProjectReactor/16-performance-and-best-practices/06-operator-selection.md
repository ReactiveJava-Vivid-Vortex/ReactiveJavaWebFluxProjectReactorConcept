# Operator Selection

## In Simple Terms

Choosing the *right* operator for a given task isn't just about correctness — it also
has real performance implications. A few common examples of operator choices that
matter:

## Simple Example

**map() vs flatMap()** — don't use `flatMap()` for synchronous transformations:

```java
// Unnecessary overhead: flatMap() is for ASYNC transformations
flux.flatMap(item -> Mono.just(transform(item)));

// Correct and more efficient: map() for synchronous transformations
flux.map(item -> transform(item));
```

**flatMap() concurrency control** — unbounded concurrency can overwhelm downstream
systems:

```java
// Potentially unbounded concurrent calls - risky against a rate-limited API
flux.flatMap(item -> callExternalApi(item));

// Bounded concurrency - much safer
flux.flatMap(item -> callExternalApi(item), 10); // max 10 concurrent calls
```

**concatMap() vs flatMap()** — use `concatMap()` when strict ordering matters (at
the cost of concurrency):

```java
flux.concatMap(item -> processInOrder(item)); // sequential, ordered
flux.flatMap(item -> processInOrder(item));   // concurrent, unordered
```

## Why It Matters

Picking the wrong operator can silently hurt performance (unnecessary
async-wrapping overhead), correctness (unordered output when order was assumed), or
stability (unbounded concurrency overwhelming a downstream dependency). Being
deliberate about operator choice — not just picking whichever "seems to work" — is a
hallmark of production-quality reactive code.
