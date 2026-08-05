# GZIP Compression — Topic Overview

## What Is This Topic About? (In Simple Terms)

This is one of the simplest, highest-value performance wins available: shrink your
HTTP response bodies with gzip before sending them over the network, and let the
client decompress them on arrival. For JSON-heavy APIs (lots of repeated field
names across many objects), this can shrink responses by 60-80% — a small amount of
CPU time traded for a much bigger reduction in bytes transferred.

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
    min-response-size: 1024 # skip compressing tiny responses — not worth the overhead
```

Once enabled, this requires **zero changes** to your controller/handler code — it's
purely a server configuration setting, applying uniformly to matching response
types above the size threshold.

The chain of benefits is straightforward: smaller responses → less bandwidth used
(lower cloud egress costs) → less time spent transferring data over the network →
faster perceived response times, especially noticeable for clients on slower or
metered connections (mobile users, for example).

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Compression** | Enable via `server.compression.enabled=true` — a single config setting, no code changes needed. |
| 2 | **Reduced bandwidth** | Smaller responses transmitted — real savings on cloud egress costs and client data usage. |
| 3 | **Faster responses** | Less data to transfer over the network usually outweighs the small compression/decompression CPU cost. |

## How It All Fits Together

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

This is a "turn it on and mostly forget about it" optimization — just remember it
pairs well with the earlier HTTP/2 topic, since both target reducing the overhead of
moving data over the network.
