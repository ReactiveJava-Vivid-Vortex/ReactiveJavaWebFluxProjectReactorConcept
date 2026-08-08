# Q1. Does Returning a Flux Automatically Mean My Endpoint "Streams"?

## Simple Explanation (Think of a Delivery Truck vs a Conveyor Belt)

Surprisingly, **no.** By default, a WebFlux endpoint returning `Flux<T>` still
waits for the **entire truck to be loaded** (the whole `Flux` to complete) before
driving off (writing the response) — it just writes one big JSON array at the end.

```java
// Default: waits for the WHOLE Flux, THEN writes one big JSON array — a delivery truck
@GetMapping("/products")
public Flux<ProductDto> getAllProducts() { return repository.findAll().map(...); }

// Streaming: each item leaves the moment it's ready — a conveyor belt
@GetMapping(value = "/products/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> streamProducts() { return repository.findAll().map(...); }
```

The **media type** is what actually flips the switch from truck to conveyor belt.

---

## Q2. What Is NDJSON, and Why Does It Enable True Streaming?

```json
[
  {"id": "1", "name": "Widget"},
  {"id": "2", "name": "Gadget"}
]
```
vs
```
{"id": "1", "name": "Widget"}
{"id": "2", "name": "Gadget"}
```

A normal JSON array can't be safely parsed until the closing `]` arrives — forcing
a client to wait for everything. **NDJSON** (newline-delimited JSON) — each line is
an independently-parseable object — lets a client start processing the instant the
first line arrives.

---

## Q3. How Does This Apply to File Uploads/Downloads?

```java
// Upload: streams bytes DIRECTLY to disk — never holds the whole file in memory
public Mono<String> uploadFile(@RequestPart("file") Mono<FilePart> filePartMono) {
    return filePartMono.flatMap(fp -> fp.transferTo(Path.of("/uploads", fp.filename()))
        .then(Mono.just("Uploaded: " + fp.filename())));
}

// Download: reads the file in small chunks, streamed to the response
public Mono<Void> downloadFile(ServerHttpResponse response) {
    Flux<DataBuffer> fileStream = DataBufferUtils.read(path, response.bufferFactory(), 4096);
    return response.writeWith(fileStream);
}
```

Both keep server memory usage **constant**, regardless of file size — critical for
multi-gigabyte files.

---

## Q4. How Do I Export Millions of Database Rows Without Running Out of Memory?

```java
@GetMapping(value = "/products/export", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> exportAllProducts() {
    return productRepository.findAll() // streamed FROM the database, not pre-loaded into a List
        .map(ProductMapper::toDto);
}
```

Because the R2DBC repository itself returns a `Flux` (not a pre-materialized
`List`), rows flow from database → server → client incrementally, never all
sitting in server memory at once.

**The one thing that would break this:** accidentally calling `.collectList()`
somewhere in the chain — that forces the entire dataset into memory before
anything is emitted.

---

## Q5. Interview-Style Q&A

### If I return `Flux<T>` with the default `application/json` media type, does the client get data incrementally?

**No** — the client waits for the entire response (one JSON array), just like a
non-streaming endpoint. You need NDJSON (or SSE) to actually stream.

### Is NDJSON the same thing as Server-Sent Events?

**No** — NDJSON is a generic streaming JSON format for any client. SSE
(`text/event-stream`) is browser-specific, consumed via the `EventSource` API —
covered in its own dedicated topic.

### What's the biggest risk when streaming millions of records?

Accidentally introducing a `.collectList()` or similar "gather everything first"
operator somewhere in the chain, which silently defeats the whole point of
streaming.

---

## Q6. Summary

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

### One sentence to remember

> **"Flux<T> alone doesn't stream — it's the media type (NDJSON or SSE) that
> turns a delivery truck (wait for everything) into a conveyor belt (send as
> it's ready)."**
