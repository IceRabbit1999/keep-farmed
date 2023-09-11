# Crust of Rust
## [Smart Pointers and Interior Mutability](https://www.youtube.com/watch?v=8O0Nt9qY_vo)
### Cell
- `Cell` type allows you to modify a value through a shared reference because:
  -  no other threads have reference to it(`!Sync`)
  -  you've never given out a reference into the value you store(return a `Copy` not a reference)

### RefCell

## [Send, Sync and their implementors](https://www.youtube.com/watch?v=yOezcP-XaIw&list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa&index=14)

# Lifetime

- [Understanding Lifetime in Rust](https://mobiarch.wordpress.com/2015/06/29/understanding-lifetime-in-rust-part-i/)
  - Lifetime solved two nagging problems: Memory management and Race condition
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