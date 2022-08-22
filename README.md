# Tartan C Enum

Support for C-style enums that support unknown values.

```rust
#
c_enum! {
    pub enum Example(u16) {
        Foo,
        Bar = 2,
        Quux,
        Baz = 0xffff,
    }
}

// Known value
let x = Example::Bar;
assert_eq!(x, Example::from(2));
assert_eq!(u16::from(x), 2);
assert_eq!(x.name(), Some("Bar"));

// Omitted variant values assigned in sequence
assert_eq!(u16::from(Example::Foo), 0);
assert_eq!(u16::from(Example::Quux), 3);

// Unknown value
let y = Example::from(0xcafe);
assert_eq!(u16::from(y), 0xcafe);
assert_eq!(y.name(), None);

// Use in an FFI-safe struct
#[repr(C)]
#[derive(Debug, PartialEq)]
pub struct Quux(Example, u8, u8);
unsafe {
    assert_eq!(
        core::mem::transmute::<[u8; 4], Quux>([0xff, 0xff, 0x8c, 0xf2]),
        Quux(Example::Baz, 0x8c, 0xf2),
    );
    assert_eq!(
        core::mem::transmute::<[u8; 4], Quux>([0xab, 0xab, 0x05, 0x3b]),
        Quux(Example::from(0xabab), 0x05, 0x3b),
    );
}
```

Rust's `enum` types trigger undefined behavior when they are assigned unknown
discriminant values (e.g., through a pointer cast or transmutation). While this
enables useful complier optimizations, it also means that `enum`s are not safe for use
in FFI, since C treats enums as integral types that can take any value within range of
the underlying integer type.

This crate offers an alternative kind of enumeration which is more similar to C.
Enumerations defined with the `c_enum` macro are simple wrappers for an integer type.
Known variants are defined as constants, and can be associated with their names
defined in code (e.g., for `Debug` output), but unknown values are fully supported.
Since they have transparent representations, they do not trigger undefined behavior
when transmuting from arbitrary values (as long as you use a built-in integer type)
and are safe to use in FFI structs and functions.

## Installation

Add to your Cargo.toml:
```
[dependencies]
tartan-c-enum = 0.1.0
```

## Development

This is a pretty standard Rust library using Cargo.

### Tests

```
cargo test --all-targets
```

### Formatting/Linting

```
cargo fmt
cargo clippy --all-targets
```

## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.

---

<small>This README was generated from doc comments. Use `cargo readme` to refresh it.</small>
