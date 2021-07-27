# What is the Java concurrency problem?
## How the project loom can fix it
### Current concurrency problem on Java
When we write a server-side application, we need our application to support high throughput, and it means we must respond to lots of requests in a small unit of time. We need to handle requests concurrently and paralleled since concurrency in java means threads we need lots of threads.
We could have a million open sockets in the new modern servers, but we can't have a million threads.
Since Java only wraps OS threads, we have as many threads as OS has, and even modern OSs can't support more than a few thousand active threads at a time.
This is our first problem; the second problem is even worse; there is a lot of context switching between threads, and they are so inefficient.
Let me describe it with an example.
Assume that we have an X value shared between thread A and thread B, and thread A wants to write to X; after that, thread B intends to read the X.
Thread A lock X and write to it the value of X stored in the CPU cache.
Then thread B wants to read the X . It can read it from the cache, but unfortunately, OS kernel schedules thread B on the other CPU core, and it can't access the thread A cache, so thread B must reread value memory, which is inefficient. So far, we have two problems;
limited threads and blind scheduling. 
So we need two things first, more threads; second, schedule threads somewhere that can predicate which line will read or write the same value.
### Loom Project
The loom project is trying to solve the above problems with a new concept in Java named virtual thread (Fiber), introducing a lightweight concurrency construct to Java. 
### Virtual Threads
Java threads are warping OS threads and very dependent on them. Virtual threads are not mapped one-to-one like normal threads. The JVM maps many virtual threads (even millions) onto a small set of OS threads, and OS doesn't know about virtual threads. They are managed internally in the Java run time. So virtual threads are cheap and unlimited. So far, the first problem solved endless threads could handle larger throughout. Since all virtual threads schedule in the Java runtime, there is more efficiency in their context switching.
