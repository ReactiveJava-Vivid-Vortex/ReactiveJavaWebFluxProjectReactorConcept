# Blocking I/O

## In Simple Terms

**I/O** just means talking to something outside the CPU/RAM — reading a file,
calling a database, making a network request. All of that is slow compared to
what the CPU can do on its own.

**Blocking I/O** means: when your thread asks for some data, it just **stops and
waits right there** until the data shows up. It can't do anything else in the
meantime — it's frozen solid, holding a spot for nothing.

## Simple Example

```java
public class BlockingIODemo {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("example.com", 80);
        InputStream in = socket.getInputStream();

        System.out.println("About to read... thread will freeze here");
        int data = in.read(); // FREEZES until data arrives over the network
        System.out.println("Got data: " + data);
    }
}
```

If the server takes 3 seconds to answer, this thread does absolutely nothing
useful for those 3 seconds. It's just parked.

## Why It Matters for Reactive Programming

If a web server gives one thread to each request, and every request makes a
blocking database call, that thread (and the memory it's holding) is wasted for
the whole wait. With enough requests at once, you run out of threads — even
though the CPU itself is barely doing anything. This is exactly the problem
non-blocking I/O and reactive programming were built to fix.
