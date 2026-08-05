# Blocking I/O

## In Simple Terms

**I/O** (Input/Output) refers to operations like reading a file, calling a database,
or making a network request — anything that talks to something *outside* the CPU/RAM,
which is comparatively very slow.

**Blocking I/O** means: when your thread asks for data (e.g., "give me the next bytes
from this socket"), it **pauses and waits** right there until the data arrives. The
thread cannot do anything else during that time — it's literally frozen, consuming a
thread "slot" for nothing.

## Simple Example

```java
public class BlockingIODemo {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("example.com", 80);
        InputStream in = socket.getInputStream();

        System.out.println("About to read... thread will freeze here");
        int data = in.read(); // BLOCKS until data arrives over the network
        System.out.println("Got data: " + data);
    }
}
```

If the server takes 3 seconds to respond, this thread does absolutely nothing useful
for those 3 seconds — it's just parked, waiting.

## Why It Matters for Reactive Programming

If a web server uses one thread per incoming request, and each request makes a
blocking database call, then during that database wait, the thread (and the memory it
holds) is wasted. With enough concurrent requests, you run out of threads even though
the CPU itself is mostly idle. This is precisely the problem non-blocking I/O and
reactive programming were designed to solve.
