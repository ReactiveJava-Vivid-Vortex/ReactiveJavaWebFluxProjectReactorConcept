# Exchange Strategies

## In Simple Terms

`ExchangeStrategies` configure how `WebClient` encodes/decodes HTTP message bodies
— things like maximum in-memory buffer size for a response, or custom
encoders/decoders for non-standard content types. The most commonly adjusted setting
is the default in-memory buffer size limit for reading response bodies.

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

Without this adjustment, attempting to fully buffer a response body larger than the
default limit (256KB) throws a
`DataBufferLimitException` — a common surprise when calling APIs that return large
JSON payloads.

## Why It Matters

Understanding `ExchangeStrategies` (and specifically the default in-memory buffer
size limit) helps you avoid a common, confusing error when `WebClient` calls fail
against APIs returning larger-than-expected responses — and lets you configure
custom encoding/decoding behavior when integrating with non-standard external APIs.
