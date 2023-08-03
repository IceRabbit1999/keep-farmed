# Rust by Question

Documenting the annoying problems in learning rust, and convincing people I'm good at it

## Popped Question

1. dyn `traitA` vs impl `traitA`
    + TL, DR: A given impl Trait cannot be created from several, distinct types, you must treat it as a single type in a given context. [Source](https://users.rust-lang.org/t/difference-between-returning-dyn-box-trait-and-impl-trait/57640/3), Another **trade-off question,** actually.
    + Pros and cons: [Source](https://www.ncameron.org/blog/dyn-trait-and-impl-trait-in-rust/)

2. zero-cost abstraction
    + TL, DR: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better. [Source](https://boats.gitlab.io/blog/post/zero-cost-abstractions/)
    + Example: ownership and borrowing -> memory safety without gc


## Make the borrower checker happy 😉

## In Books

1. `affine typing`
    + The definition on wiki: Affine types are a version of linear types allowing to discard (i.e. not use) a resource, corresponding to affine logic. **An affine resource can be used at most once**, while a linear one must be used exactly once.
    + Sounds like something ownership/borrower checker does
2. `gdb/lldb`, `valgrind`, `massif`

## Theoretical

### Rust主要特性
[Source](https://www.youtube.com/watch?v=njle3PQIwUg)
- 内存安全
  - 所有权是什么？为什么重要？借用是什么？有什么作用？
  - 生命周期是什么？有什么作用？
- 并发性
- 零成本抽象
- 零开销异常处理
- 模式匹配和函数式编程

### 经典面试题
[Source](https://rustcc.cn/article?id=0b0afa3e-db03-428e-9fc5-b06347997d41)
- RwLock<T>对想要在多线程下正确使用，T的约束是？
  - The type parameter T represents the data that this lock protects. It is required that T satisfies Send to be shared across threads and Sync to allow concurrent access through readers
- 下面代码能否通过编译？为什么？
```rust
trait A{ fn foo(&self) -> Self; }
Box<Vec<dyn A>>;
//不可以
```
- `Clone`和`Copy`的区别
  - Copy是marker trait，告诉编译器需要move的时候copy。Clone表示拷贝语义，有函数体。不正确的实现Clone可能会导致Copy出BUG
  - Copy is implicit and an inexpensive bit-wise copy, while Clone is always explicit and may or may not be expensive
  - Rust does not allow you to reimplement Copy
- `deref`的被调用过程？
  - Deref 是一个trait，由于rust在调用的时候会自动加入正确数量的 * 表示解引用。则，即使你不加入*也能调用到Deref
- Rust里如何实现在函数入口和出口自动打印一行日志？
  - 调用处宏调用、声明时用宏声明包裹、proc_macro包裹函数、邪道一点用compiler plugin、llvm插桩等形式进行
- `Box<dyn (Fn() + Send +'static)>`是什么意思?
  - 一个可以被Send到其他线程里的没有参数和返回值的callable对象，即 Closure，同时是 ownershiped，带有static的生命周期，也就说明没有对上下文的引用

### Box
A data type that allows you to store a value on the heap
- why would you want to store data on the heap instead of the stack?[Source](https://anooppoommen.medium.com/understanding-box-in-rust-5d5164e554e)
  - heap is generally much larger than the stack(e.g., a vector contains a million elements )
  - data stored on the heap can be shared between different parts of a program(e.g., threads, recursive data structures like linked list)
- when to use? [Source](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html#using-boxt-to-point-to-data-on-the-heap)
  - When you have a type whose size can’t be known at compile time
  - When you have a large amount of data
  - When you want to own a value, and you care only that it’s a type
- box vs reference [Source](https://stackoverflow.com/questions/36117057/when-to-use-box-instead-of-reference)
  - box:  denotes that a type is **owned** and that it is allocated on the heap
  - reference: denotes that you are borrowing the value from something else
  - another way to think: 
    - if your data does not have a place to live in, put it in a box to give it a home
    - if you know that the data you want to work with has a home to live in: you can usually just use a reference (address) to just get access to it 

### RC
Short for reference counting: enables **shared ownership** of a value, and be deallocated only when the last pointer is dropped
- why need rc? 
  - a single value might have multiple owners in real world
- Rc::clone
```rust
// Rc clone
fn clone(&self) -> Rc<T> {
   Increment reference count
   self.inner().inc_strong();
// Generate a new Rc structure via self.ptr
   Self::from_inner(self.ptr)
}
```
Rc’s clone doesn’t copy the actual data, it just adds a reference count.
- usage
  - 多线程共享所有权
  - GUI编程
  - 递归数据结构：图、树

### Cell/RefCell
Both provide `Interior Mutability`: where you can modify the value stored in cell via immutable reference of the cell.
- difference [Source](https://blog.iany.me/2019/02/rust-cell-and-refcell/)
  - Cell: 
    - **copies** or **moves** contained value
    - Cell cannot tell you what's contained in the cell via a reference
  - RefCell
    - allows both mutable and immutable reference borrowing. It tracks the borrows at **runtime**, via borrow and borrow_mut.
    - runtime tracking has **overheads** and RefCell can lead to runtime panics
    - RefCell just **defers** the borrowing rule from compile time to run time, not bypass

### Deref
Implementing the Deref trait allows us to customize the behavior of the dereference operator: *
- usage [Source](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch15-02-deref.html#how-deref-coercion-interacts-with-mutability)
  - `T: Deref<Target=U>`: &T -> &U
  - `T: DerefMut<Target=U>`: &mut T -> &mut U
  - `T: Deref<Target=U>`: &mut T -> &U
- no runtime overheads because the number of times that `Deref::deref` needs to be inserted is resolved at compile time
- It is recommends to implement Deref trait to smarter pointers only

### Drop
The Drop trait provides a way to run some code when a value goes out of scope.
- what's for [Source](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/drop.html#drop)
  - Drop is used to clean up any resources associated with a struct

### Clone/Copy
- difference: [Source](https://doc.rust-lang.org/std/clone/trait.Clone.html)
  - Differs from Copy in that Copy is implicit and an inexpensive bit-wise copy, while Clone is always explicit and may or may not be expensive.
  - Rust does not allow you to reimplement Copy, but you may reimplement Clone and run arbitrary code.

### Any
A trait to emulate dynamic typing(The opposite of static typing)
- use cases? [Source](https://www.reddit.com/r/rust/comments/j770uf/what_use_cases_are_there_for_the_any_trait/)
  - New Rust Programmer: use Any because it allows them to pretend that Rust supports dynamic typing.
  - Intermediate Rust programmer: avoid Any because they know it leads to problems when abused.
  - Advanced Rust programmer: use Any because it lets them create awesome type-safe interfaces that bend the rules of what static type systems can achieve

### Send/Sync
Send: ownership of values of the type implementing Send can be transferred between threads

Sync: it is safe for the type implementing Sync to be referenced from multiple threads
- define [Source](https://doc.rust-lang.org/nomicon/send-and-sync.html#send-and-sync)
  - A type is Send if it is safe to send it to another thread.
  - A type is Sync if it is safe to share between threads (T is Sync if and only if &T is Send).
- Major exceptions include:
  - raw pointers are neither Send nor Sync (because they have no safety guards).
  - UnsafeCell isn't Sync (and therefore Cell and RefCell aren't).
  - Rc isn't Send or Sync (because the refcount is shared and unsynchronized).
- use cases [Source](https://www.zhihu.com/question/303273488)
  - Send but not Sync: 编译期保证无法用于多线程share data的场景, 实现了Send可以把所有权move到另一个线程独占使用
  - Sync but not Send: MutexGuard
- difference and commonality [Source1](https://www.zhihu.com/question/303273488) [Source2](https://huonw.github.io/blog/2015/02/some-notes-on-send-and-sync/)
  - Send表示跨线程move, Sync表示跨线程share data，基本是ownership和borrower的区别
  - 对于复合类型当且仅当所有成员都是Send/Sync时，这个类型才是Send/Sync
  - Sync is related to how a type works when shared across multiple threads at once, and Send talks about how a type behaves as it crosses a task boundary
  - Sync + Copy => Send: if a type T implements both Sync and Copy, then it can also implement Send (conversely, a type is only allowed to be both Sync and Copy if it is also Send).
  - &mut T: Send when T: Send

### Fn/FnOnce/FnMut
Fn: A closure which has an immutable context belongs to Fn (&self)

FnMut: A closure which has a mutable context belongs to FnMut (&mut self)

FnOnce: A closure that owns its context belongs to FnOnce (self)

- relationship [Source](https://stackoverflow.com/questions/30177395/when-does-a-closure-implement-fn-fnmut-and-fnonce)
  - Closures will implement them automatically if it seems to be needed
  - All closures implement FnOnce

