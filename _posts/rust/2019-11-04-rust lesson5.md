---
title: "Lesson 05-function"
date: 2019-11-06 18:22:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C05-%ED%95%A8%EC%88%98)
<br/>

The basic usage of functions is similar to JavaScript, but in Rust, `fn` is used instead of `function`.

```rust
fn main() {}
```

## 1. Parameters

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

In Rust, it is mandatory to define the type of parameters. Failure to do so will result in an error, a point that JavaScript developers should note carefully. Type annotations are crucial for Rust's strong typing system.

## 2. Return Values

In Rust, returning values differs from JavaScript. The `return` keyword can be omitted, and the semicolon (`;`) should also be removed. This creates an expression-based return.

```rust
fn five() -> i32 {
    5
}

fn plus_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

In `fn five() -> i32`, `i32` indicates the return type of the function. Defining return types enables compile-time checks for type consistency, enhancing code reliability.

## 3. Attributes

- Conditional Execution

Using `#[cfg(condition)]` allows a function to be executed only when the specified condition is met.

The example below is executed only on Windows; otherwise, an error occurs.

```rust
fn main() {
    on_windows();
}

#[cfg(target_os = "windows")]
fn on_windows() {
    println!("This OS is windows.")
}

// Result - when the OS is Windows:
// This OS is windows.

// Result - when the OS is Linux:
// |
// 2 |  on_windows();
// |  ^^^^^^^^^^ not found in this scope
```

- Testing

Used with `#[test]`, this attribute tests whether a function can be executed.

You can test with `rustc` or `cargo` as follows:

```bash
rustc --test [filename]
```

Or:

```bash
cargo test
```

We will cover the `#[...]` syntax in more detail later. Attributes provide powerful mechanisms for conditional compilation and testing in Rust.