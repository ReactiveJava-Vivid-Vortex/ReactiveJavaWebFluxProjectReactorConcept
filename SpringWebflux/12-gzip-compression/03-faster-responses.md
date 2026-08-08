# Faster Responses

## In Simple Terms

Since compressed responses are smaller, they take less time to travel over
the network — even factoring in the small extra bit of CPU time needed to
compress and decompress them. For most JSON APIs, the time saved on
transfer easily outweighs the compression overhead, so the response feels
faster to the client overall.

## Simple Example

Rough, illustrative comparison for a client on a moderate-speed connection:

```
Without compression:
  250 KB response over a 1 Mbps connection -> ~2 seconds transfer time

With gzip compression:
  35 KB response (same data, compressed) -> ~0.3 seconds transfer time
  + ~10-20ms compression/decompression overhead
  -> net significant improvement in perceived response time
```

The benefit shows up most on slower networks (mobile connections) and with
bigger response payloads — for tiny responses (a few hundred bytes), the
compression overhead might not even be worth it, which is why
`min-response-size` (see [[compression]]) exists to skip compressing very
small responses.

## Why It Matters

Faster responses are a real, noticeable improvement to how snappy an app
feels — and they cost almost nothing to turn on (usually one config
setting), with no changes needed to your actual API logic or client code.
