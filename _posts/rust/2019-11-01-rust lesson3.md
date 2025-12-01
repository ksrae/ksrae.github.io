---
title: "Lesson 03-Variables"
date: 2019-11-01 11:00:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C03-%EB%B3%80%EC%88%98)
<br/>

## 1. Immutable vs. Mutable Declarations

```rust
let x: i32 = 1;
// Type declaration can be omitted. If omitted, the compiler infers and applies the type.
```

By default, when you define `x` using `let`, the standard variable declaration method in Rust, `x` becomes immutable (a value that cannot be changed). Consequently, you cannot modify its value after declaration, such as `x = 2;`.

In essence, the default declaration behaves as if it's `readonly`.

```rust
let x = 1;
x = 2;
// error: cannot assign twice to immutable variable `x`
```

To create a mutable variable, you must declare it with `mut`. Adding `mut` after `let` makes the variable mutable, allowing you to change its value.

```rust
let mut x = 1;
x = 2;
// x is now 2
```

However, you cannot change the variable's type. `mut` only removes the `readonly` restriction; it does not allow changing the variable's type.

```rust
// Valid: Shadowing (re-declaration with the same name)
let spaces = "   ";
let spaces = spaces.len(); // spaces is now a usize (integer)

// Invalid: Type mismatch
let mut spaces = "   ";
spaces = spaces.len(); // error: mismatched types
```

### Exception: Shadowing

An exception exists where you can "change" a value without `mut` by re-declaring it with the same name (shadowing), provided the new declaration has the same type. This effectively re-assigns the value in the same memory location (or a new location depending on compiler optimizations).

```rust
let x = 1;
let x = x + 1;
let y = 5;
let y = x + 2;
```

## 2. Scope

Similar to JavaScript, variable scope is restricted. Even for immutable variables, if the scope differs, you can declare another variable with the same name.

```rust
fn main() {
    let outer = 1;
    {
        let outer = 2;
        let inner = 3;
        println!("{} {}", outer, inner); // Output: 2 3
    }
    println!("{}", outer); // Output: 1
}
```

In this case, unlike the re-declaration mentioned above, Rust recognizes these as entirely different variables stored in different memory locations. (According to Ownership, which we will cover later, variables go out of scope and are dropped, removing them from memory.)

## 3. Important Considerations

Unlike JavaScript, assignment expressions do not cascade from left to right. It is recommended to declare one variable per line for clarity and to avoid potential errors.

```rust
// Invalid: Assignment within an assignment is not allowed.
// let x = (let y = 5);  // Error

// Suggested: Separate declarations for clarity.
let y = 5;
let x = y;
```

## 4. Values and Addresses

If you want a variable to share the same memory address as another variable, similar to pointers in C, you should use a reference (`&`) when assigning the value.

```rust
fn main() {
    let t1 = 10;
    let t2 = &t1; // t2 is a reference to t1
    println!("{:p} {:p}", &t1, t2);
}

// Output (Addresses will vary)
// 0x77114ff5f4 0x77114ff5f4
```

If you declare a variable as a copy (without `&`), a copy of the value is created, and the variables do not share the same memory address.

```rust
fn main() {
    let t1 = 10;
    let t2 = t1; // t2 is a copy of t1
    println!("{:p} {:p}", &t1, &t2);
}

// Output (Addresses will vary)
// 0x5e368ff820 0x5e368ff824
```

Rust allows you to express both the value and the address based on the output formatting options, without requiring pointer dereferencing using `*` like in C. You can use format specifiers like `{:p}` to display memory addresses.