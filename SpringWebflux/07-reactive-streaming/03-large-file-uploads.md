# Large File Uploads

## In Simple Terms

WebFlux can handle big file uploads reactively — processing the file's
bytes as a `Flux<DataBuffer>` incrementally, instead of loading the whole
file into memory before it even starts writing it somewhere.

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

`FilePart.transferTo()` streams the incoming file straight to its
destination without ever holding the whole thing in memory at once —
critical if you want to accept multi-gigabyte uploads without risking an
`OutOfMemoryError`.

## Why It Matters

Large file uploads are a textbook case where reactive streaming really
pays off: memory usage stays roughly flat no matter how big the file is,
since data flows through the pipeline (network → processing → disk)
incrementally instead of piling up at any one point.
