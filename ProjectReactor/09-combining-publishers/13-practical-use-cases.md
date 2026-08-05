# Practical Use Cases (Combining Publishers)

## In Simple Terms

Combining operators aren't just academic — they map directly onto extremely common
real-world microservice patterns. Here are three of the most frequent.

## 1. Cache + Database

Try a fast cache first, fall back to the database only if the cache misses:

```java
public Mono<Product> getProduct(String id) {
    return cache.get(id)
        .switchIfEmpty(
            database.findById(id)
                .doOnNext(product -> cache.put(id, product)) // populate cache
        );
}
```

## 2. Multiple Backend Calls (Fetched Concurrently)

Fetch a user's profile and their recent orders **in parallel**, then combine:

```java
public Mono<Dashboard> getDashboard(String userId) {
    Mono<UserProfile> profileMono = userService.getProfile(userId);
    Mono<List<Order>> ordersMono = orderService.getRecentOrders(userId);

    return Mono.zip(profileMono, ordersMono)
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()));
}
```

This takes roughly as long as the **slower** of the two calls, instead of the sum of
both (which sequential blocking code would incur).

## 3. Aggregating Responses from Several Microservices

```java
public Flux<SearchResult> searchAllSources(String query) {
    Flux<SearchResult> fromServiceA = serviceAClient.search(query);
    Flux<SearchResult> fromServiceB = serviceBClient.search(query);

    return Flux.merge(fromServiceA, fromServiceB); // combine, don't care about order
}
```

## Why It Matters

These three patterns — fallback (`switchIfEmpty`), parallel combination (`zip`), and
aggregation (`merge`) — cover the vast majority of real inter-service composition
needs in reactive microservices, and knowing which operator maps to which pattern
saves a lot of trial-and-error when designing a new endpoint.
