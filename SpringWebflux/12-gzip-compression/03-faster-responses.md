# Faster Responses

## In Simple Terms

Because compressed responses are smaller, they take less time to transmit over the
network — even accounting for the small additional CPU time needed to compress and
decompress the data. For most JSON APIs, the network-transfer time saved
significantly outweighs the compression overhead, resulting in a faster overall
response, as perceived by the client.

## Simple Example

Rough illustrative comparison, for a client on a moderate-speed connection:

```
Without compression:
  250 KB response over a 1 Mbps connection -> ~2 seconds transfer time

With gzip compression:
  35 KB response (same data, compressed) -> ~0.3 seconds transfer time
  + ~10-20ms compression/decompression overhead
  -> net significant improvement in perceived response time
```

The benefit is most pronounced on slower networks (mobile connections) and for
larger response payloads — for tiny responses (a few hundred bytes), compression
overhead might not be worth it, which is why `min-response-size` (shown in
[[compression]]) exists to skip compressing very small responses.

## Why It Matters

Faster responses directly improve user-perceived application performance — a real,
measurable benefit that costs very little to enable (usually a single configuration
setting) and requires no changes to your actual API logic or client code.
