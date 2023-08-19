# Lifetime

1. from [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)
    - `T` only contains owned types ❌
      - `T` is a superset of both `&T` and `&mut T`
      - `&T` and `&mut T` are disjoint sets
    - if `T: 'static` then `T` must be valid for the entire program ❌

# Async
## [Learning Async Rust With Entirely Too Many Web Server](https://ibraheem.ca/posts/too-many-web-servers/)
1. We already have threads, so why async?
  - It makes it very difficult to model two operations: **cancellation, selection**
  - Expressing event-based logic becomes very difficult when kernel holds so much control over the execution of our program
  - We lose control of our program during I/O and have no choice but to wait until it completes. There's really no good way of force canceling a thread
2. The biggest problem non-blocking IO brings
  - An efficient way to keep track all of our active connections, and somehow get notified when they become ready. On Linux, it's called **epoll**  
3. `Pin`: You can only create a `Pin<&mut T>` if you guarantee that the T will stay in stable location until it is dropped, meaning that any self-references will remain valid
# Popular Crates
## [Decrusting the serde crate](https://www.youtube.com/watch?v=BI_bHCGRgMY)
- basically model:  `data type --> serde data model <-- data format`
- basically steps for serialize: 
  1. `serialize_struct`
  2. `serialize_filed`
  3. `end`
- self-describe type
- serde is not a parsing library, it just provide a connection