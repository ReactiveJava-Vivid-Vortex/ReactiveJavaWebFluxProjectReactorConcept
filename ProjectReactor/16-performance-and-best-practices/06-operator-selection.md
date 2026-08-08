# Operator Selection

## In Simple Terms

Picking the *right* operator for the job isn't just about getting the
right answer — it also affects performance. Here are a few common choices
that actually matter.

## Simple Example

**map() vs flatMap()** — don't reach for `flatMap()` for plain, synchronous
changes:

```java
// Unnecessary overhead: flatMap() is for ASYNC transformations
flux.flatMap(item -> Mono.just(transform(item)));

// Correct and more efficient: map() for synchronous transformations
flux.map(item -> transform(item));
```

**flatMap() concurrency control** — letting concurrency run wild can
overwhelm whatever's downstream:

```java
// Potentially unbounded concurrent calls - risky against a rate-limited API
flux.flatMap(item -> callExternalApi(item));

// Bounded concurrency - much safer
flux.flatMap(item -> callExternalApi(item), 10); // max 10 concurrent calls
```

**concatMap() vs flatMap()** — reach for `concatMap()` when strict order
matters (you trade away concurrency to get it):

```java
flux.concatMap(item -> processInOrder(item)); // sequential, ordered
flux.flatMap(item -> processInOrder(item));   // concurrent, unordered
```

## Why It Matters

Picking the wrong operator can quietly hurt performance (extra
async-wrapping overhead for no reason), correctness (results coming back
out of order when you assumed they wouldn't), or stability (unbounded
concurrency overwhelming something downstream). Being deliberate about
which operator you reach for — not just whatever happens to compile — is
what separates production-quality reactive code from code that just seems
to work.
