---
title: "Lesson 08-Reference and Borrowing"
date: 2019-11-19 17:16:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C08-Reference-and-Borrowing)
<br/>

Let's delve into References and Borrowing in Rust.

```rust
fn main() {
    let s = String::from("hello");
    ownership(s);

    println!("s length {}", s.len());
}

fn ownership(s: String) {
    println!("{}", s);
}
```

In the above code snippet, attempting to execute `s.len()` results in an error. This is because ownership of the `String` `s` has been transferred to the `ownership` function.

**Reference**

In JavaScript, arguments can be passed to functions either by reference or by value. Let's investigate these two approaches in the context of Rust.

- By Reference

```rust
fn main() {
    let s = String::from("hello");
    let s1 = byreference(&s);
    println!("{}", s1);
}

fn byreference(s: &String) -> (&String) {
  s
}
```

When passing by reference, only the memory address is passed, so ownership is not transferred. Consequently, the original variable remains accessible after the function call.

- By Value

```rust

fn main() {
    let s = String::from("hello");
    let s1 = byvalue(s);
    println!("{}", s1);
}

fn byvalue(s: String) -> (String) {
  s
}
```

Passing by value transfers ownership of the data. Therefore, after the function call, the original variable is dropped and its memory reclaimed.

**Mutable Reference**

In both of the preceding cases, the arguments were immutable, preventing modifications within the function. To address this, we have two primary solutions:

- Assign the value to a new variable within the function.
- Pass the argument as a mutable reference (`&mut`).

Let's examine the second approach. To pass a mutable reference, the original variable must itself be declared as mutable.

Consider the following example:

```rust
fn main() {
    let mut s = String::from("hello");
    let s1 = byreference(&mut s);

    println!("{}", s1);
}

fn byreference(s_mut: &mut String) -> (&String) {
    s_mut.push_str(", world");
    s_mut
}
```

The `&` symbol is a special character that enables referencing. The opposite operation can be performed using `*`, similar to its usage in C.

### Borrow

In the examples above, `&mut` is used for borrowing rather than directly referencing a memory address. This constitutes a distinct mechanism compared to simple referencing with `&`.

- Ownership: The state where a variable no longer resides in memory.
- Borrow: A mutable reference. The variable exists, but ownership is temporarily transferred to another variable or function. This typically occurs within a specific scope, indicated by `&mut`.
- Reference: An immutable reference. Allows accessing a variable's value without affecting ownership.

Multiple immutable references are permissible:

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
```

However, borrowing is disallowed while an immutable reference is still active:

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &mut s;
// Error occurs
```

Borrowing is allowed after the immutable reference is no longer in use:

```rust
let mut s = String::from("hello");

let r1 = &s;
println!("{}", r1);

let r2 = &mut s;
// OK
```

Borrowing is permissible even after passing a reference to a function:

```rust
fn main() {
    let mut s = String::from("hello");
    print(&s);

    let r3 = &mut s;
    println!("s {}", r3);
}

fn print(s: &String) {
    let r1 = &s;
    println!("r1 {}", r1);
}
```

However, if a function returns a reference, borrowing becomes impossible:

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = print(&s);

    let r3 = &mut s;
    println!("s {} {}", r1, r3);
}

fn print(s: &String) -> (&String) {
    let r1 = &s;
    r1
}

// Error
```

Returning a value, instead of a reference, enables borrowing:

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = print(&s);

    let r3 = &mut s;
    println!("s {} {}", r1, r3);
}

fn print(s: &String) -> (String) {
    let r1 = &s;
    r1.to_string()
}

// Success
```

Conversely, while borrowing is active, creating a reference is also disallowed:

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = print(&mut s);

    let r3 = &s;
    println!("s {} {}", r1, r3);
}

fn print(s: &mut String) -> (&mut String) {
    s
}
// Error
// |
//3 |     let r1 = print(&mut s);
// |                    ------ mutable borrow occurs here
//4 |
//5 |     let r3 = &s;
// |              ^^ immutable borrow occurs here
//6 |     println!("s {} {}", r1, r3);
// |                         -- mutable borrow later used here
```

Similarly, borrowing is disallowed while another borrow is active:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{} {}", r1, r2);
}

// Result
// |
//4 |     let r1 = &mut s;
// |              ------ first mutable borrow occurs here
//5 |     let r2 = &mut s;
// |              ^^^^^^ second mutable borrow occurs here
//6 |
//7 |     println!("{} {}", r1, r2);
// |                       -- first borrow later used here
```

In summary:

- reference -> reference: OK
- reference -> borrow: Disallowed
- borrow -> reference: Disallowed
- borrow -> borrow: Disallowed
- After reference/borrow ends -> reference/borrow: OK

### Dangling References

References and borrows become invalid when the underlying lifetime expires.

Returning a reference to a variable created within a function is unsafe because the original variable is dropped when the function exits, rendering the reference invalid.

```rust
fn main() {
    let r1 = dangling();

    println!("{}", r1);
}

fn dangling() -> (&String) {
    let s = String::from("hello");
    &s
}
// Result
// |
//7 | fn dangling() -> (&String) {
// |                   ^ help: consider giving it a 'static lifetime: `&'static`
// |
//  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed
```

In such cases, you must return the value directly:

```rust
fn main() {
    let r1 = dangling();

    println!("{}", r1);
}

fn dangling() -> (String) {
    let s = String::from("hello");
    s
}
```

## Summary of References

- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.