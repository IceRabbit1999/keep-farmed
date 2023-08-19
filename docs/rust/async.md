- How can we arrange for `wake()` to be called once the socket becomes readable?
  - This problem is solved through integration with an IO-aware system blocking primitive, such as `epoll` on linux

- `async/.await` make it possible to yield control of the current thread rather than blocking
- `async fn/async block` returns a value that implements the `Future` trait
- A `Future` is an asynchronous computation that can produce a value
- Any variables used in `async` bodies must be able to travel between threads when using a multithreaded `Future` executor
- `Pinning` makes it possible to guarantee that an object implementing `!Unpin` won't ever be moved

