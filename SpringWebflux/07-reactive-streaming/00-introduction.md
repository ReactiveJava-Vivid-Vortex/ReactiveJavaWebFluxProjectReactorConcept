# Reactive Streaming — Topic Overview

## What Is This Topic About? (In Simple Terms)

By default, a WebFlux endpoint returning `Flux<T>` still waits for the **entire**
`Flux` to complete before writing anything — it just writes one big JSON array at
the end. This topic is about actually streaming: sending each item to the client
**as soon as it's ready**, using a streaming media type like NDJSON, instead of
buffering everything first.

```java
// Default: waits for the WHOLE Flux, then writes one big JSON array
@GetMapping("/products")
public Flux<ProductDto> getAllProducts() { return repository.findAll().map(...); }

// Streaming: writes each item as soon as it's available
@GetMapping(value = "/products/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> streamProducts() { return repository.findAll().map(...); }
```

**NDJSON** (newline-delimited JSON) is the key format enabling this — each line is
an independently-parseable JSON object, so a client can start processing results the
instant the first line arrives, rather than waiting for a closing `]` bracket that
only appears at the very end of a normal JSON array.

This same streaming principle applies in both directions: **large file
uploads/downloads** stream bytes incrementally (never loading a whole file into
memory), and a reactive **repository** returning `Flux<Entity>` means you can export
millions of database rows while keeping server memory usage roughly constant
throughout — as long as you avoid accidentally collecting everything into a `List`
first.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Server Streaming** | Endpoint sends response data incrementally as it's produced, via a streaming media type. |
| 2 | **Client Streaming** | Endpoint accepts a request body that itself streams in incrementally (e.g., `Flux<Dto>` upload). |
| 3 | **Large File Uploads** | `FilePart.transferTo()` streams bytes directly to disk — never buffers the whole file in memory. |
| 4 | **Large File Downloads** | `DataBufferUtils.read()` streams a file to the response in small chunks, constant memory usage. |
| 5 | **JSON Lines (NDJSON)** | Newline-delimited JSON — each line independently parseable, ideal for incremental streaming. |
| 6 | **Streaming Millions of Records** | Reactive repository (`Flux`) + NDJSON = export huge datasets with roughly constant server memory. |
| 7 | **text/event-stream** | The MIME type for Server-Sent Events — server-to-client push, covered in depth in the SSE topic. |
| 8 | **Memory-efficient processing** | Avoid `.collectList()` on huge streams — use `.reduce()`/incremental processing to keep memory usage bounded. |

## How It All Fits Together

```
Data source (DB rows, file bytes, upload body)
        │
        ▼
Keep it as a Flux — DON'T .collectList() it (unless truly needed)
        │
        ▼
Return/consume with a streaming media type:
    - APPLICATION_NDJSON_VALUE  → generic streaming JSON records
    - TEXT_EVENT_STREAM_VALUE   → browser-facing live push (SSE topic)
        │
        ▼
Data flows incrementally, server memory stays roughly constant
regardless of total dataset/file size
```

The one habit to build from this topic: **before returning `Flux<T>` from an
endpoint, ask "should this actually stream?"** — if the dataset could be large or
slow to produce, NDJSON (or SSE, for browsers) is almost always the better choice
over the default buffered JSON array.
