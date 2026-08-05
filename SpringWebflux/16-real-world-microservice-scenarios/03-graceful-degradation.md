# Graceful Degradation

## In Simple Terms

"Graceful degradation" means a system continues to provide **reduced but still
useful** functionality when parts of it fail, rather than failing completely. This
is a broader principle than partial responses — it applies to any strategy for
maintaining partial value under failure: fallback data, cached results, simplified
responses, or reduced features.

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

Even if the fancy ML-powered search service is down, users still get functional
(if less sophisticated) search results from a simpler, more reliable fallback
service — rather than a broken search feature entirely.

## Why It Matters

Graceful degradation directly embodies the Reactive Manifesto's "Resilient" trait
([[resilient]]) in a very concrete, user-facing way — designing every critical
feature with a fallback plan (even a simplified one) means real failures translate
into "slightly worse experience" rather than "broken feature" for end users.
