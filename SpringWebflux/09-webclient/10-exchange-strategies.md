# Exchange Strategies

## In Simple Terms

`ExchangeStrategies` control how `WebClient` reads and writes HTTP message
bodies — things like the maximum amount of a response it'll hold in
memory, or custom encoders/decoders for unusual content types. The setting
people run into most often is the default in-memory buffer size limit for
reading response bodies.

## Simple Example

```java
ExchangeStrategies strategies = ExchangeStrategies.builder()
    .codecs(configurer -> configurer
        .defaultCodecs()
        .maxInMemorySize(10 * 1024 * 1024) // increase limit to 10MB (default is 256KB)
    )
    .build();

WebClient webClient = WebClient.builder()
    .baseUrl("https://api.example.com")
    .exchangeStrategies(strategies)
    .build();
```

Without this adjustment, trying to fully buffer a response body bigger
than the default 256KB limit throws a `DataBufferLimitException` — a
common surprise when calling APIs that return large JSON payloads.

## Why It Matters

Knowing about `ExchangeStrategies` (especially the default in-memory
buffer limit) saves you from a confusing error when `WebClient` calls fail
against APIs returning bigger-than-expected responses — and lets you set
up custom encoding/decoding when talking to non-standard external APIs.
