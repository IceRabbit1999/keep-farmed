# Lifetime

1. from [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)
    - `T` only contains owned types ❌
      - `T` is a superset of both `&T` and `&mut T`
      - `&T` and `&mut T` are disjoint sets
    - if `T: 'static` then `T` must be valid for the entire program ❌
      - 