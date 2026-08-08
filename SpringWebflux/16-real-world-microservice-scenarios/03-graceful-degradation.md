# Graceful Degradation

## In Simple Terms

"Graceful degradation" means a system keeps offering reduced-but-still-useful
functionality when parts of it break, instead of failing outright. It's a
broader idea than partial responses — it covers any way of holding onto
partial value under failure: fallback data, cached results, simplified
responses, or fewer features.

## Simple Example

```java
public Mono<SearchResults> search(String query) {
    return advancedSearchService.search(query) // ML-powered, feature-rich search
        .timeout(Duration.ofSeconds(1))
        .onErrorResume(error -> {
            log.warn("Advanced search unavailable, falling back to basic search", error);
            return basicSearchService.search(query); // simpler but reliable fallback
        });
}
```

Even if the fancy ML-powered search is down, users still get working (if
less fancy) search results from a simpler, more reliable fallback —
instead of search breaking entirely.

## Why It Matters

Graceful degradation puts the Reactive Manifesto's "Resilient" trait
([[resilient]]) into practice in a very real, user-facing way — giving
every important feature a fallback plan (even a simplified one) means
failures turn into "slightly worse experience" instead of "broken
feature" for the people using it.
