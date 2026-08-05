# Reduced Bandwidth

## In Simple Terms

"Reduced bandwidth" is the direct, measurable benefit of enabling compression —
less data physically transmitted over the network for the same logical response
content. This matters both for server-side network costs (many cloud providers
charge for egress bandwidth) and client-side experience (faster downloads,
especially on constrained connections).

## Simple Example

Illustrative comparison for a JSON API response listing 1,000 products:

```
Uncompressed response size: ~250 KB
Gzip-compressed response size: ~35 KB   (roughly 85% smaller)
```

The exact compression ratio depends heavily on the data's structure — repetitive
JSON (similar field names repeated across many objects) compresses extremely well,
while already-compressed data (images, video) sees little to no benefit from gzip.

## Why It Matters

Reduced bandwidth translates directly into lower cloud hosting costs (many providers
bill for data egress) and faster response delivery to end users — particularly
impactful for APIs serving large collections or verbose JSON structures, and for
clients on slower or metered network connections.
