# Q1. What Is GZIP Compression, and Why Is It "Free" Performance?

## Simple Explanation (Think of Vacuum-Sealing a Suitcase Before a Flight)

You don't make your clothes lighter by vacuum-sealing them — you just make them
take up **less space in transit**. GZIP does exactly this to an HTTP response:
shrink the bytes before shipping them over the network, unpack them on arrival.

```
Uncompressed:  ~250 KB response for a JSON list of 1,000 products
GZIP-compressed: ~35 KB   (roughly 85% smaller — repetitive JSON compresses VERY well)
```

The "cost" is a small amount of CPU time to compress/decompress — almost always
worth it for text-heavy responses like JSON.

---

## Q2. How Do I Turn It On?

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
    min-response-size: 1024 # skip tiny responses — not worth the overhead
```

**Zero controller/handler code changes needed** — it's purely a server
configuration setting, applying automatically to matching response types above
the size threshold.

---

## Q3. Why Skip Tiny Responses? (`min-response-size`)

Compressing a 50-byte response might actually make it *bigger* once compression
headers are added, and the CPU overhead isn't worth it for something already
small. `min-response-size` sets a floor below which compression is skipped
entirely.

---

## Q4. What's the Chain of Benefits?

```
Smaller response
     │
     ├──▶ Reduced bandwidth (lower cloud egress costs)
     │
     └──▶ Faster responses (especially on slow/mobile connections)
              — the time saved transferring fewer bytes usually far outweighs
                the small compression/decompression CPU cost
```

---

## Q5. Interview-Style Q&A

### Does compression help binary data like images the same way?

**No** — already-compressed formats (JPEG, MP4, ZIP) see little to no benefit;
GZIP shines on repetitive text (JSON, XML, HTML).

### Do I need to change my `Mono`/`Flux` response types to enable compression?

**No** — it's applied at the HTTP transport layer, completely independent of your
reactive pipeline code.

### Does GZIP pair well with HTTP/2?

**Yes** — they attack the same problem (network overhead) from different angles:
compression shrinks payload size, HTTP/2 shrinks connection/header overhead.

---

## Q6. Summary

```
Response body (JSON, XML, text) larger than min-response-size?
        │
        ▼  (enabled via server.compression.enabled=true)
gzip-compress the response body
        │
        ▼
Smaller payload sent over the network
        │
        ├──▶ Reduced bandwidth (lower egress costs)
        └──▶ Faster responses (especially on slow/mobile connections)
```

### One sentence to remember

> **"GZIP is vacuum-sealing your HTTP responses — turn it on with one config
> setting, and JSON-heavy APIs typically shrink by 60-80% for a small CPU
> cost."**
