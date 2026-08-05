# Partial Responses

## In Simple Terms

When aggregating data from multiple downstream services, sometimes one service
fails while others succeed. Rather than failing the entire response, a well-designed
reactive service can return a **partial response** — the data that's available,
with a clear indication of what's missing.

## Simple Example

```java
public record Dashboard(
    Optional<UserProfile> profile,
    Optional<List<Order>> recentOrders,
    Optional<Recommendations> recommendations
) {}

public Mono<Dashboard> getDashboard(String userId) {
    Mono<Optional<UserProfile>> profileMono = userService.getProfile(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    Mono<Optional<List<Order>>> ordersMono = orderService.getRecentOrders(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    Mono<Optional<Recommendations>> recsMono = recommendationService.getFor(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    return Mono.zip(profileMono, ordersMono, recsMono)
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2(), tuple.getT3()));
}
```

If the recommendation service is down, the dashboard still renders with the profile
and order data — just with `recommendations` empty — rather than failing the entire
request.

## Why It Matters

Partial responses provide a much better user experience than an all-or-nothing
approach: a dashboard missing one non-critical widget is far better than a completely
blank error page, especially when the failing dependency is non-essential to the
overall page.
