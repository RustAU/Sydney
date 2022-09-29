---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# **Conditional Compilation in Rust**

- Tobin Harding (me@tobin.cc)

---

## What in this talk

- very briefly describe the build tools and attributes
- what is the `cfg` attribute and how to use it
- what is a configuration option and what forms it can take
- how to set configuration options
- what is the `cfg!` macro
- bench marking use case

---

## Build tools

- `cargo` is the Rust package manager
- `rustc` is the Rust compiler
- You can use `cargo` to interact with `rustc`

---

## Attributes

- Rust has attributes
- Attributes are metadata for the compiler

---

## The cfg attribute

```
#[cfg(foo)]
{
    // This gets built in if foo is set
}
```

---

## Configuration options

- `foo` is a conditional "configuration option"
- It has two forms

- `#[cfg(foo)]`
- `#[cfg(foo = "bar")]`

'foo' is key, '"bar"' is a value

---

## cfg helpers

There are three helpers `any`, `all`, `not`

- `#[cfg(any(foo, bar))]`
- `#[cfg(all(foo, bar))]`
- `#[cfg(not(foo))]`

Can be nested arbitrarily.

---

## Setting configuration options

Configuration options can be called anything. The can be set by:

- `cargo test` sets the `test` config option
- `cargo --features=foo` sets the `feature = "foo"` config option
- `RUSTFLAGS='--cfg=bar' cargo check` sets the `bar` config option
- `rustc --cfg=foo` sets the `foo** config option

---

## Conditional attributes

**Warning terms are overloaded in this slide**

There are various attributes as well as `cfg`, for example

```rust
#[feature(test)]
```

Attributes can be conditionally included using `cfg_attr`

- `#[cfg_attr(foo, feature(test))]` 

Is equivalent to `#[features(test)]` iff `foo` is set.

---

## Branching

As well as conditionally compiling you can branch using `cfg!` macro

```rust
    if cfg!(foo) {
    	println!("foo is set");
    }
```

---

## Overloading the word 'test'

```rust
#[cfg(test)]
mod tests {

    #[test]
    fn it_works() {
        // test code
    }
}
```

- `test` is a configuration option set by cargo
- module name is `tests` by convention
- `#[test]` is an attribute

---

## More about features

- Remember `feature` is not special other than it is set by cargo
- features should be additive
- `cargo test --all-features` should run if all features are additive

---

## Benchmark use case

- bench marking can require use of an unstable crate called `test`
- we can't use the `test` crate with a stable toolchain
- we can't hide the `test` crate behind a feature because features are additive 
- The recommended use of `"unstable"` feature violates the last point
- we can use `RUSTFLAGS='--cfg=bench' cargo bench` to run benches along with

---

> The internals of the test crate are unstable, behind the test flag

```rust
#![cfg_attr(bench, feature(test))]

#[cfg(bench)]
extern crate test

...

#[cfg(bench)]
mod bench {
    // bench mark code here.
}
```

---

## What I've told you

- what `cargo` is, what `rustc` is and what an attribute is
- what the `cfg` attribute is and how to use it
- what a configuration option is and its two forms
- how to set configuration options
- what is the `cfg!` macro
- a way to conditionally run benchmarks while maintaining `cargo test --all-features`
