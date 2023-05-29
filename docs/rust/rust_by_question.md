# Rust by Question

Documenting the annoying problems in learning rust, and convincing people I'm good at it

## Popped Question

1. dyn `traitA` vs impl `traitA`
    + TL, DR: A given impl Trait cannot be created from several, distinct types, you must treat it as a single type in a given context. [Source](https://users.rust-lang.org/t/difference-between-returning-dyn-box-trait-and-impl-trait/57640/3), Another **trade-off question,** actually.
    + Pros and cons: [Source](https://www.ncameron.org/blog/dyn-trait-and-impl-trait-in-rust/)

2. zero-cost abstraction
    + TL, DR: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better. [Source](https://boats.gitlab.io/blog/post/zero-cost-abstractions/)
    + Example: ownership and borrowing -> memory safety without gc


## Make the borrower checker happy :)

## In Books

1. `affine typing`
    + The definition on wiki: Affine types are a version of linear types allowing to discard (i.e. not use) a resource, corresponding to affine logic. **An affine resource can be used at most once**, while a linear one must be used exactly once.
    + Sounds like something ownership/borrower checker does
2. `gdb/lldb`, `valgrind`, `massif`

## For Interview

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
