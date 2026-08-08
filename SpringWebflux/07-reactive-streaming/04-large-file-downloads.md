# Large File Downloads

## In Simple Terms

The same idea works in reverse for downloads: WebFlux can stream a big
file to the client incrementally — reading its bytes as a
`Flux<DataBuffer>` and writing them straight into the HTTP response as
they're read, without ever loading the whole file into server memory.

## Simple Example

```java
@GetMapping("/files/{filename}")
public Mono<Void> downloadFile(@PathVariable String filename, ServerHttpResponse response) {
    Path filePath = Path.of("/uploads", filename);

    response.getHeaders().set(HttpHeaders.CONTENT_DISPOSITION,
        "attachment; filename=\"" + filename + "\"");

    Flux<DataBuffer> fileStream = DataBufferUtils.read(
        filePath, response.bufferFactory(), 4096 // read in 4KB chunks
    );

    return response.writeWith(fileStream);
}
```

The file gets read from disk and written to the response in small chunks,
continuously — the server never holds more than a small buffer's worth of
the file in memory at any moment, no matter how big the file actually is.

## Why It Matters

Streaming downloads keeps server memory usage flat and predictable, even
when serving huge files (videos, backups, datasets) to lots of clients at
once — a naive "load the whole file into a byte array first" approach
risks running the server out of memory under concurrent large downloads.
