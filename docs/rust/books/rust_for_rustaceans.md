# Foundations
## Memory Terminology
- A value is stored in a place (a location that can hold a value)|
- This place can be on the stack, on the heap...
- The most common place is a variable: a named value slot on the stack

## Memory Regions
### Stack
- The stack is a segment of memory that your program uses as scratch space for function calls
- Each time a function is called, a contiguous chunk of memory called a frame is allocated at the top of the stack
- A function's frame contains all the variables within that function, along with any arguments the function takes
- When the function returns, its stack frame is reclaimed

### Heap
- The heap is a pool of memory that isn't tied to the current call stack of the program
- Values in heap memory live until they are explicitly deallocated

### Static Memory
there can be 'static references that do not point to static memory

## Interior Mutability

- Get a mutable reference through a shared reference: `Mutex`, `RefCell`
- Let you replace a value given only a shared reference: `Cell`, `std::sync::atmoic`

## Lifetime Variance

wtf is this:

```rust
struct MutStr<'a, 'b> {
    s: &'a mut &'b str }
let mut s = "hello";
*MutStr { s: &mut s }.s = "world";
println!("{}", s)
```

# Types

Types tell you how to interpret bits of memory

## Types in Memory

### Alignment

- You can create a pointer pointing only to byte 0 or byte 1(8bits)
- For this reason, all values must start at a byte boundary (at least byte-aligned: be placed at an address that is a multiple of 8bits)
- On a 64-bit CPU, most values are accessed in chunks of 8 bytes(64bits). This is referred to the CPU's word size
- Operations on data that is not aligned are referred to misaligned accesses, which can lead to poor performance and bad concurrency problems 
- Complex types that contain other types are typically assigned the largest alignment of any type they contain

### Layout

```rust
#[repr(C)]
struct Foo {
    tiny: bool, // 1 byte
    normal: u32, // 4 bytes
    small: u8, // 1 byte
    long: u64, // 8 bytes
    short: u16 // 2 bytes
}
```

C layout: for a 64-bits CPU: 1 + 3(padding) + 4 + 1 + 7(padding) + 8 + 2 = 26bytes + 6 = 32 bytes

Rust layout: 8 + 4 + 2 + 1 + 1 = 16bytes

### Dynamically Sized Types and Wide Pointers

Two common types that are not `Sized`: trait objects and slices

Wide pointer, aka Fat pointer, is twice the size of a `usize`: one `usize` for holding the pointer and one `usize` for holding the extra information needed to "complete" the type

## Traits and Traits Bounds

### Compilation and Dispatch

- To be object-safe: 
  - non of a trait's method can be generic or use the `Self` type
  - the trait cannot have any static methods
- `Self: Sized` implies that `Self` is not being used through a trait object

### Generic Traits

- `trait<T> Foo` vs `trait Foo { type Bar }` : use an associated type if you expert only one implementation of the trait for a given type

# Error Handling

## Representing Errors

### Enumeration













