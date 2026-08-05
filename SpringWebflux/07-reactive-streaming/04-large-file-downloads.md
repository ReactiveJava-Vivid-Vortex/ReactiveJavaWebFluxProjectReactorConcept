# Large File Downloads

## In Simple Terms

Similarly, WebFlux can stream large file **downloads** to a client incrementally —
reading the file's bytes as a `Flux<DataBuffer>` and writing them directly to the
HTTP response as they're read, without ever loading the entire file into server
memory.

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

The file is read from disk and written to the HTTP response in small chunks,
streamed continuously — the server never holds more than a small buffer's worth of
the file in memory at any given moment, regardless of the total file size.

## Why It Matters

Streaming large downloads keeps server memory usage constant and predictable, even
when serving very large files (videos, backups, datasets) to many concurrent
clients simultaneously — a naive "load the whole file into a byte array first"
approach would risk exhausting server memory under concurrent large downloads.
