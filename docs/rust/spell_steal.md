# Crust of Rust

## Async/Await
- yield until something happens
- every await is a opportunity for whoever is above you to choose to do something else
- if you don't await it, it doesn't get to do anything. awaiting is what drives its progress
- a way to do cancellation
- async just describes the mechanisms for changing or for cooperatively scheduling a bunch of computation by describing when under what circumstances code can make progress and under what circumstances code can yield
- you are only allowed to await in async functions and async blocks
- top-level future that describes the entire control flow of your application, something has to like run that in a loop. `main()` can't yield because there's noting above
- the executor crate(tokio) provides both the lowest level resources(network, sockets, timers) and also the executor loop at the top and the two are sort of wired up together behind the scenes
- cooperatively scheduled which means other things get to run as long as the thing that's running occasionally yields. so if you have a future which uses something like `std::io`, `std::net`, that thread is block, and now non of your futures get to make any progress, they're never polled again
- the way that you can introduce parallelism not just concurrency is `tokio::spawn()`
- introduce parallelism into asynchronous programs is you need to communicate the futures that can run in parallel to the executor
- `fn foo() -> impl Future<Output = ()>`: the `impl Future<Output = ()>` as actually a kind of **StateMachine**
- the StateMachine can become very large because it will hold all "stateful" variables... in all futures, the solution is to use `Box`, and that's also a reason to use `tokio::spawn` 
- why can't `async trait`: the type of the "thing" that async trait produces isn't known and written anywhere, the size of it is depends on the implementation. (the size of the StateMachine)
- you can use a `std::sync::Mutex` as long as your critical section is short and not contain any await points and yield points

## Lifetime Annotations
source: https://www.youtube.com/watch?v=rAl-9HwD858&t=3061s
- if you have any lifetime you can assign to it anything of the same type but a longer lifetime

## Decrusting the tracing crate
source: https://www.youtube.com/watch?v=21rtHinFA40
- the application side which is emits events, subscriber side which is the thing that receives or acts upon events
- a span is essentially a sort of logical grouping of related events

## Iterators
source: https://www.youtube.com/watch?v=yozQ9C69pNs&t=3237s


## Smart Pointers and Interior Mutability
source: https://www.youtube.com/watch?v=8O0Nt9qY_vo
- there is no way with cell for you to get a reference to what's inside the cell, and so it's always safe to mutate it
- `Cell` does not implement `Sync`: if you have a reference to a cell, you can not give away that reference to a different thread
- `UnsafeCell` lists itself as the core primitive for interior mutability in rust
- `RefCell` lets you check at runtime whether anyone else is mutating


## [Smart Pointers and Interior Mutability](https://www.youtube.com/watch?v=8O0Nt9qY_vo)
### Cell
- `Cell` type allows you to modify a value through a shared reference because:
  -  no other threads have reference to it(`!Sync`)
  -  you've never given out a reference into the value you store(return a `Copy` not a reference)

### RefCell

# Lifetime

## RustLifetimes
source: https://www.youtube.com/watch?v=1QoT9fmPYr8

- lifetimes enforce a piece of memory is still valid for a reference
- lifetimes are really about insuring memory does not get cleaned up before a reference can use it
- rust uses the minimum of the passed in lifetimes
- 


# Framework

## [Dependency Injection In Axum](https://tulipemoutarde.be/posts/2023-08-20-depencency-injection-rust-axum/)
### static dispatch
```rust
pub trait AppState: Clone + Send + Sync + 'static {
  type D: database::DB;

  fn db(&self) -> &Self::D;
  fn templates(&self) -> &Handlerbars<'static>;
}

#[derive(Clone)]
pub struct RegularAppState {
  pub templates: Handlerbars<'static>,
  pub db: MemoryDB,
}

impl AppState for RegularAppState {
  type D = MemoryDB;
  
  fn db(&self) -> Self::D {
    &self.db
  }

  fn templates(&self) -> &Handlerbars<'static> {
    &self.templates
  }
}
```

```rust
pub fn build_router() -> Router {
  let state = RegularAppState {
    templates: build_templates(),
    db: MemoryDB::new(),
  };
  build(state)
}

fn build<A: AppState(state: A) -> Router {
  Router::new()
    .route("/", get(handlers::index::<A>))
    .route("/item/:id", get(handlers::show::<A>))
    .with_state(state)
}
```
```rust
pub async fn index<A: AppState>(State(state): State<S>) {
  
}

pub async fn show<S: AppState>(Path(id): Path<Uuid>, State(state): State<S>) -> Html<'static> {

}
```
Two drawbacks:
- Every combination of dependencies need a new struct that implements `AppState`
- No way to directly match again the actual fields in the handler definition
### dynamic dispatch
```rust
#[derive(Clone)]
pub struct AppState {
  pub templates: Handlerbars<'static>,
  pub db: Arc<dyn DB + Send + Sync>,
}

pub fn build_router() -> Router {
  let state = AppState {
    templates: build_templates(),
    db: Arc::new(MemoryDB::new()),
  };

  Router::new()
    .route("/", get(handlers::index))
    .route("/item/:id", get(handlers::show))
    .with_state(state)
}
```

```rust
pub async fn index(State(AppState {db, templates}): State<AppState>) -> Html<String> {

}

pub async fn show(Path(id): Path<Uuid>, State(AppState {db, templates}): State<AppState>) -> Html<String> {}
```

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

## Axum
source: Decrusting axum
- How Router works?
  - A `service` that takes a request and returns a response
- How your random functions implement `Handler`?
  - The `Handler` trait is automatically implemented for a lot of types. magic macro: `impl_handler`, `all_the_tuples`
  - A function is automatically implemented Handler if:
    - The last function parameter implements `FromRequest` and the other function parameters implement `FromRequestParts`
    - The function matches: `FnOnce(T1, T2 ...) -> Fut + ... where Fut: Future<Output=Res> where Res: IntoResponse`
- Why the last argument implement `FromRequest`
  - the last argument consumes the body
- The moment you provide the state to a `Handler`, it turns into a `Service`


## [Decrusting the serde crate](https://www.youtube.com/watch?v=BI_bHCGRgMY)
- basically model:  `data type --> serde data model <-- data format`
- basically steps for serialize: 
  1. `serialize_struct`
  2. `serialize_filed`
  3. `end`
- self-describe type
- serde is not a parsing library, it just provide a connection

## [Rayon: Data Parallelism](https://www.youtube.com/watch?v=gof_OEv71Aw)
- Parallel iterators
  - closures cannot mutate shared state
  - some operations are different(fold, find)
- rayon::join()
  - add `join` wherever parallelism is possible
  - let the library decide when it is profitable(how many threads to use)

# Rust itself

## The What and How of Futures and async/await in Rust
source: https://www.youtube.com/watch?v=9_3krAQtD2k&t=4s

- lazy of future: just describing the steps that it's going to go through, and this is where the notion of an executor comes in
- executor is basically something that you can give futures to and it will make sure that they get done
- if Pending is returned, it must has arranged for yourself to be woken up again when you can make progress
- the tricky part is to find some way to have these tasks be notified
- futures define sort of general principles for or the general interface for how to deal with asynchronous computation like tasks and notifiers and wake up and executors, it only define the interfaces
- tokio on the other hand is an implementation of both an executor and sort of the other infrastructure that you need in order to make things wake up
- where tokio shines is it also gives you the thing that is going to wake you up: reactor
- reactor/epoll persuade code
```rust
enum Operation {
  Read,
  Write
}
// may not in another thread
fn reactor_thread(notify_me: mpsc::Receiver<(Task, FD, Operation)>) {
  let waiting_for = HashMap<(FD, Operation), Task>;
  loop {
    while let Some((task, fd, op)) = notify_me.try_recv() {
      waiting_for.insert((fd, op), task);
    }
    let select = waiting_for.keys.collect();
    for (fd, op) in epoll(select) {
      waiting_for.remove((fd, op), task).notify();
    }
  }
}
```
- persuade code on Pending
```rust
struct Foo {
  fd: std::net::TcpStream
}
impl Future for Foo {
  fn poll(&mut self) -> Result<(Poll, Error)> {
    loop {
      match self.fd.read() {
      Ok(r) => {
        // read success
      },
      Err(io::Error::WouldBlock) => {
        // must do something to make sure we are woken up!
        let reactor = Handle::current();
        // use epoll actually
        reactor.register(self.fd, Operation::Read, task::current());
        return Ok(Pending)
      },
      Err(io::Error::Closed) => {
        return Ok(Ready)
      },
      Err(e) => {}
    }
    }
  }
}
```
- tokio runtime is basically a thing that provides you with a reactor, an executor and a timer
- if you have tons of futures and they're all ready, you basically need to find the right way to switch between dealing with the futures and driving the reactor or driving the timer
- 
## Printing Things The Wrong Way
[source](https://endler.dev/2023/cursed-rust/)
Ways to print `Hello, world`
1. Desugaring `println!`: `write!(std::io::lock(), "Hello, world!")`
2. Iterating over characters: `"Hello, world!".chars().map(|c| print!("{c}))`
3. Impl `Display`:
```rust
struct HelloWorld;

impl std::fmt::Display for HelloWorld {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "Hello, world!")
    }
}

println!("{HelloWorld}");
```
4. Build your own `Display`
```rust
trait Println {
    fn println(&self);
}

impl Println for &str {
    fn println(&self) {
        print!("{}", self);
    }
}

"Hello, world!".println();
``` 
5. panic!: `panic!("Hello, world!);`
6. Closures: `(|s: &str| print!("{s}"))("hello"); // call a closure directly after its definition`
7. C style:
```rust
extern crate libc;
use libc::{c_char, c_int};
use core::ffi::CStr;

extern "C" {
    fn printf(fmt: *const c_char, ...) -> c_int;
}

fn main() {
    const HI: &CStr = match CStr::from_bytes_until_nul(b"hello\n\0") {
        Ok(x) => x,
        Err(_) => panic!(),
    };

    unsafe {
        printf(HI.as_ptr());
    }
}
```
8. C++ style of course
9. Threads:
```rust
fn main() {
    let phrase = "hello world";
    let phrase = Arc::new(Mutex::new(phrase.chars().collect::<Vec<_>>()));

    let mut handles = vec![];

    for i in 0..phrase.lock().unwrap().len() {
        let phrase = Arc::clone(&phrase);
        let handle = thread::spawn(move || {
            thread::sleep(Duration::from_millis(((i + 1) * 100) as u64));
            print!("{}", phrase.lock().unwrap()[i]);
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
    println!();
}
```
# Language Level

## Rust Is Beyond Object-Oriented
source: https://www.thecodedmessage.com/posts/
### Encapsulation
- class-based encapsulation is not some special insight of OOP, but a specialized - or rather, tightly restricted form of module
- in most cases, `getter` and `setter` is unnecessary because it is not raw field accessing(a network call or something else), it is just simple records. In other words: I should hope `foo.bar = 3` would never make a network call
- in reality, `get` and `set` functions are only used as wrappers to satisfy the constraints of object-oriented ideology
### Polymorphism
- in OOP, polymorphism tries to take all decision-making in a common narrow mechanism: runtime polymorphism, and here are some constrains
  - indirection: dereferencing the object reference, dereferencing the class pointer/vtable pointer, doing the indirect function call
  - the call is indirect means that inlining is impossible
  - it is polymorphic in one parameter only
  - each value is independently polymorphic
  - entangled with other OOP features
- having a mechanism of vtable-based runtime polymorphism isn't a bad thing
- 4 alternatives in Rust that can be used when OOP uses runtime polymorphism
  - enum
    ```
    // Java
    class UserId {
        // ...
        public abstract bool isAdministrator();
    }

    class Username extends UserId {
        // ...
        public bool isAdministrator() {
            return username.startsWith("admin_");
        }
    }

    class AnonymousUser extends UserId {
        // ...
        public bool isAdminstrator() {
            return false;
        }
    }
    // Rust
    fn is_administrator(user: &UserId) -> bool {
      match user {
          UserId::Username(name) => name.starts_with("admin_"),
          UserId::AnonymousUser(_) => false,
      }
    }
    ```
  - closures: an OOP interface or polymorphic decision only involves one actual operation, in such a situation, a closure ca just be used instead
    - if you find yourself writing a trait with exactly one method, consider whether you should instead be using some sort of closure or lambda type
  - polymorphism with traits
  - dynamic trait objects: you have a closed set of operations that can be performed on a value, but what those operations actually do will change dynamically in a way that cannot be bounded ahead of time
### Inheritance
- some parent classes have fields, and that's when inheritance really starts to have problems
- if you have both fields and virtual methods you have true OOP-style inheritance, with all of its problems
- it is like in order to implement a trait, your type must have certain fields
- rather than conflate the concepts of record types, modules, and traits in this God-concept of "class", Rust keeps these three concepts quite separate