# Practical Use Cases (Combining Publishers)

## In Simple Terms

Combining operators aren't just theory — they map straight onto patterns
you'll actually use all the time in real services. Here are three of the
most common.

## 1. Cache + Database

Try a fast cache first, and only bother the database if the cache doesn't
have it:

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

Fetch a user's profile and their recent orders **at the same time**, then
combine the two:

```java
public Mono<Dashboard> getDashboard(String userId) {
    Mono<UserProfile> profileMono = userService.getProfile(userId);
    Mono<List<Order>> ordersMono = orderService.getRecentOrders(userId);

    return Mono.zip(profileMono, ordersMono)
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()));
}
```

This takes roughly as long as the *slower* of the two calls, instead of
adding both together the way sequential blocking code would.

## 3. Aggregating Responses from Several Microservices

```java
public Flux<SearchResult> searchAllSources(String query) {
    Flux<SearchResult> fromServiceA = serviceAClient.search(query);
    Flux<SearchResult> fromServiceB = serviceBClient.search(query);

    return Flux.merge(fromServiceA, fromServiceB); // combine, don't care about order
}
```

## Why It Matters

These three patterns — fallback (`switchIfEmpty`), combining parallel work
(`zip`), and gathering results together (`merge`) — cover most of the
inter-service composition you'll ever need in a reactive app. Knowing which
operator fits which situation saves a lot of guesswork when you're building
a new endpoint.
