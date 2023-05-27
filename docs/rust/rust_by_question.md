# Rust by Question

Documenting the annoying problems in learning rust, and convincing people I'm good at it

## Popped Question

1. dyn `traitA` vs impl `traitA`
    + TL, DR: A given impl Trait cannot be created from several, distinct types, you must treat it as a single type in a given context.[Copied from here](https://users.rust-lang.org/t/difference-between-returning-dyn-box-trait-and-impl-trait/57640/3), Another **trade-off question,** actually.
    + Pros and cons: [Copied from here](https://www.ncameron.org/blog/dyn-trait-and-impl-trait-in-rust/)

2. zero-cost abstraction
    + TL, DR: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better. [Copied from here](https://boats.gitlab.io/blog/post/zero-cost-abstractions/)
    + Example: ownership and borrowing -> memory safety without gc


## Make the borrower checker happy :)

## In Books

1. `affine typing`
    + The definition on wiki: Affine types are a version of linear types allowing to discard (i.e. not use) a resource, corresponding to affine logic. **An affine resource can be used at most once**, while a linear one must be used exactly once.
    + Sounds like something ownership/borrower checker does
2. `gdb/lldb`, `valgrind`, `massif`
