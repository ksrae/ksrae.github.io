---
title: "Lesson 06-control and loop"
date: 2019-11-08 18:41:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C06-%EC%A0%9C%EC%96%B4%EB%AC%B8%EA%B3%BC-%EB%B0%98%EB%B3%B5%EB%AC%B8)
<br/>

Generally, every language has control statements like `if` and `switch`, and loops like `for` and `while`.

Rust also has these commands, which we will briefly explore.

The characteristics of Rust's control statements and loops are that the conditions are not included in `()`, and the `for` statement uses `for..in..`.

Since these are basic commands, we will focus on examples.

## if

```rust
if number < 5 {
} else if {
} else {
}
```

- You cannot use an `int` type as a `bool` type.

```rust
let number = 1;
if number {
} else {
}
// error
```

You should write it in the following way:

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```

- You can put the result into a variable.

```rust
let condition = if number > 0 {
    5
} else {
    6
};
```

- At this time, the values returned from `if`, `else if`, and `else` must be of the same type.

```rust
let condition = if number < 0 {
    5
} else if number == 0 {
    false
} else {
    "Positive"
};
// Each are int, bool, and string types, so it doesn't work.
```

## loop: Unlimited loop without condition

```rust
loop {
}
```

**break**

- You can put a formula into `break` and use it as a return value.

```rust
let mut counter = 0;
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};
```

## while: loop with condition

```rust
let mut index = 0;
while index < 5 {
  println!("the value is: {}", a[index]);

  index += 1;
}
```

## for

Only `for...in` from javascript is possible. If you specify a range after `in`, it can be stored in a variable declared before `in` and used in the `for` statement.

```rust
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("the value is: {}", element);
}

const MAX = 10;
for number in (1..MAX) {
  println!("{}!", number);
}

for number in (1..4).rev() {
  println!("{}!", number);
}
```

`i32` in `fn five() -> i32` means the return type.

## Attribute

- Conditional Execution

If you use `#[cfg(condition)]`, the function will only be executed when the condition is allowed.
The example below is code that is executed only when it is windows, and an error occurs otherwise.

```rust
fn main() {
    on_windows();
}

#[cfg(target_os = "windows")]
fn on_windows() {
    println!("This OS is windows.")
}

// Result - When the OS is windows,
// This OS is windows.

// Result - When the OS is linux,
//error[E0425]: cannot find function `on_windows` in this scope
// --> src/main.rs:2:5
//   |
// 2 |     on_windows();
//   |     ^^^^^^^^^^ not found in this scope
```

**Testing**

It is used as `#[test]` and tests whether the function can be performed.
You can test with `rustc` or `cargo` as follows.

```bash
rustc --test [file name]
```

Or

```bash
cargo test
```

We will cover the `#[...]` syntax in more detail later.