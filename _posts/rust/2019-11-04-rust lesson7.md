---
title: "Lesson 07-Ownership"
date: 2019-11-11 17:01:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C07-Ownership)
<br/>

# Understanding Ownership in Rust: Copy vs. Drop

Rust's ownership system is a cornerstone of its memory safety guarantees. Ownership refers to the variable that "owns" a piece of data in memory. When ownership is transferred, the original owner can no longer access the data. This article dives into the concepts of Copy and Drop, which are key to understanding how ownership works in Rust. We'll explore the implications of these concepts through examples and code snippets.

Ownership dictates how data is managed, impacting concepts like References and Borrowing. Grasping Copy and Drop mechanisms is crucial for writing safe and efficient Rust code.

## Shallow Copy and Deep Copy

Let's first differentiate between shallow and deep copies, concepts crucial for understanding memory management.

- **Shallow copy:** This creates a new variable that points to the *same* memory location as the original. If the underlying data is mutated through one variable, the change is visible through the other because they both reference the same memory.
- **Deep copy:**  This creates a completely new copy of the data at a *different* memory location. Modifications to one copy do *not* affect the other, providing independent data management.

In many languages, assigning a variable directly might perform a shallow copy. However, in Rust, the default behavior often leans toward moves and deep copies to ensure memory safety and prevent data races.

```rust
let s1 = "hello";
let s2 = s1;

let s3 = 10;
let s4 = &s3;

println!("{:p} {:p}", s1, s2);

//Result
// 0x7ff65b76d2e0 0x7ff65b76d2e0
```

The example above demonstrates that basic string literals (`&str`) are indeed copied, and both variables point to the same memory address. However, if the value of a string literal is modified, it will be assigned to a new address. Rust's default behavior ensures Deep Copy.

```rust
let s1 = "hello";
let mut s3 = s1;
s3 = "hallow";
println!("{:p} {} {:p} {}", s1, s1, s3, s3);

//Result
//0x7ff6b992d2e0 hello 0x7ff6b992d2e5 hallow
// Even if you change the value of s1, the result is the same.
```

When using `String::from`, the variable doesn't store the literal string directly; instead, it holds a pointer to the string data on the heap. Copying this variable directly would only copy the pointer, leading to potential double-free errors.

To perform a deep copy with `String::from`, use the `.clone()` method:

```rust
let t1 = String::from("hello");
let t2 = t1.clone();
println!("{:p} {:p}", &t1, &t2); // Because it's a variable with a value, you need to add '&' to get the address.

//Result
// 0xb0a90ff7b8 0xb0a90ff7d0
```

This creates a distinct copy of the string data in memory, ensuring that `t1` and `t2` are independent.

## Ownership and Functions

In JavaScript, passing variables to functions doesn't usually prevent the caller from using the original variable afterward. Rust's ownership system behaves differently, potentially invalidating variables after they've been passed to functions.

```rust
fn main() {
    let t1 = String::from("hello");
    str_ownership(t1);

    println!("t1 {}", t1);
}

fn str_ownership(s: String) {
    println!("str_ownership {}", s);
}

//Error
//  |
//2 |     let t1 = String::from("hello");
//  |         -- move occurs because `t1` has type `std::string::String`, which does not implement the `Copy` trait
//3 |     str_ownership(t1);
//  |                   -- value moved here
//4 |
//5 |     println!("t1 {}", t1);
//  |                       ^^ value borrowed here after move
```

In the above example, when `t1` (a `String`) is passed to `str_ownership`, ownership of the string data is *moved* to the function. Consequently, the `main` function can no longer use `t1` after the function call.

However, this behavior doesn't occur in all cases. Primitive types like integers implement the `Copy` trait.

```rust
fn main() {
    let t1 = 10;
    int_ownership(t1);

    println!("t1 {}", t1);
}

fn int_ownership(s: i32) {
    println!("int_ownership {}", s);
}

//Result
//int_ownership 10
//t1 10
```

In this scenario, passing `t1` (an `i32`) to `int_ownership` *copies* the value instead of moving ownership. Therefore, `t1` remains valid in `main` after the function call.

## Copy

The `Copy` trait dictates whether a type's value is copied when assigned or passed to a function. Let's examine the characteristics and conditions under which `Copy` occurs.

A type that implements `Copy` resides in memory even after going out of scope; `Drop` is not called. This indicates that Ownership is not affected.

The following types implement the `Copy` trait:

- All integer types, such as `u32`.
- The Boolean type, `bool`, with values `true` and `false`.
- All floating-point types, such as `f64`.
- The character type, `char`.
- Tuples, if they *only* contain types that are also `Copy`. For example, `(i32, i32)` is `Copy`, but `(i32, String)` is not.

## Drop

For types that *do not* implement `Copy` (like `String`), the data is deallocated from memory when it goes out of scope or ownership is transferred. This memory deallocation is managed by the `Drop` trait. When ownership is moved, the original variable is no longer valid.

```rust
fn main() {
    let s = String::from("hello");
    ownership(s);

    println!("s {}", s);
}

fn ownership(s: String) {
    println!("{}", s);
}

//Result
//  |
//4 |     let s = String::from("hello");
//  |         - move occurs because `s` has type `std::string::String`, which does not implement the `Copy` trait
//5 |     ownership(s);
//  |               - value moved here
//6 |
//7 |     println!("s {}", s);
//  |                      ^ value borrowed here after move
```

In contrast, types that implement `Copy` are copied, not moved. Therefore, the original variable remains valid after the function call.

```rust
fn main() {
    let s = 10;
    ownership(s);

    println!("s {}", s);
}

fn ownership(s: i32) {
    println!("{}", s);
}

//Result
//10
//s 10
```

## Exceptions

### - Copying &str

The `&str` type, representing a string slice, behaves differently.  It utilizes shallow copying, transferring only the pointer to the string data rather than the data itself.  Consequently, `&str` does not trigger the `Drop` trait.

```rust
fn main() {
    let t1 = "hello";
    str_ownership(t1);

    println!("t1 {}", t1);
}

fn str_ownership(s: &str) {
    println!("str_ownership {}", s);
}

//Result
//str_ownership hello
//t1 hello
```

### Dropping Function Return Values

Function return values go out of scope when the function ends, triggering the `Drop` trait, regardless of the type.

```rust
fn main() {
    let s = String::from("hello");
    let (s1, len) = ownership(s);
    println!("{} {}", s1, len);
}

fn ownership(s: String) -> (String, usize) {
    let slen = s.len();
    (s, slen)
}
```