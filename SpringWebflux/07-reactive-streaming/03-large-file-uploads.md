# Large File Uploads

## In Simple Terms

WebFlux can handle large file uploads reactively — processing the file's bytes as a
`Flux<DataBuffer>` incrementally, rather than loading the entire file into memory
before beginning to write it to disk (or wherever it needs to go).

## Simple Example

```java
@PostMapping("/files/upload")
public Mono<String> uploadFile(@RequestPart("file") Mono<FilePart> filePartMono) {
    return filePartMono.flatMap(filePart -> {
        Path targetPath = Path.of("/uploads", filePart.filename());
        return filePart.transferTo(targetPath) // streams directly to disk, non-blocking
            .then(Mono.just("Upload successful: " + filePart.filename()));
    });
}
```

`FilePart.transferTo()` streams the incoming file data directly to the destination
without ever holding the entire file content in memory at once — critical for
handling multi-gigabyte uploads without risking `OutOfMemoryError`.

## Why It Matters

Large file uploads are a classic case where the reactive streaming model shines:
memory usage stays roughly constant regardless of file size, since data flows
through the pipeline (network -> processing -> disk) incrementally instead of being
fully buffered at any single point.
