# Rate Limiting

## In Simple Terms

"Rate limiting" in a reactive context means deliberately controlling **how fast**
items flow through a pipeline — often to respect an external constraint, like a
third-party API's requests-per-second limit, rather than a technical inability to
keep up.

## Simple Example

Using `.delayElements()` to space out emissions:

```java
Flux.range(1, 10)
    .delayElements(Duration.ofMillis(200)) // emit at most one every 200ms
    .subscribe(n -> System.out.println("Calling API with: " + n));
```

Using `.limitRate(n)` to control how many items are requested from the source at a
time (useful for large or expensive-to-produce sources):

```java
Flux.range(1, 1_000_000)
    .limitRate(50) // request from the source in batches of 50, not all at once
    .subscribe(n -> processItem(n));
```

Combining rate limiting with concurrency control (e.g., only 5 concurrent outgoing
calls at a time):

```java
Flux.fromIterable(userIds)
    .flatMap(id -> callExternalApi(id), 5) // max concurrency of 5
    .subscribe(response -> System.out.println("Got: " + response));
```

## Why It Matters

Rate limiting is essential when your reactive pipeline interacts with external
systems that impose their own limits (API quotas, database connection pools) —
without it, a reactive pipeline could technically produce/consume data faster than
the outside world can safely handle, leading to `429 Too Many Requests` errors or
overwhelmed downstream systems.
