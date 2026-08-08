# Reduced Bandwidth

## In Simple Terms

"Reduced bandwidth" is the direct, measurable payoff of turning on
compression — less data physically going over the network for the same
content. It matters for both server costs (many cloud providers charge for
data leaving their network) and for the client's experience (faster
downloads, especially on a spotty connection).

## Simple Example

Illustrative comparison for a JSON API response listing 1,000 products:

```
Uncompressed response size: ~250 KB
Gzip-compressed response size: ~35 KB   (roughly 85% smaller)
```

The exact savings depend a lot on the data's shape — repetitive JSON
(similar field names repeated across many objects) compresses really
well, while already-compressed data (images, video) barely benefits from
gzip at all.

## Why It Matters

Less bandwidth means directly lower cloud hosting bills (many providers
charge for outgoing data) and faster delivery to end users — especially
noticeable for APIs serving large collections or verbose JSON, and for
clients on slower or metered connections.
