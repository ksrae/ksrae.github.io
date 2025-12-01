---
title: "Lesson 02-Comments and Variables types"
date: 2019-10-31 18:38:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C02-%EC%A3%BC%EC%84%9D%EA%B3%BC-%EB%B3%80%EC%88%98-%ED%83%80%EC%9E%85)
<br/>

## 1. Comments

Commenting in Rust mirrors the style found in JavaScript.

```rust
// Single-line comment
/*
Multi-line comment
*/
/// Documentation comment
```

While multi-line comments are supported, using single-line comments (`//`) is generally recommended for clarity.  For documentation purposes, utilize the `///` style comments, which are processed by Rust's documentation generator.

## 2. Global Variables

Global variables in Rust are declared using the `static` keyword. By convention, their names are written in all capital letters.

```rust
static MAX: i32 = 100;
```

String literals are typically represented using the `&str` type.

```rust
static NAME: &str = "GLOBAL VALUE";
```

Using `&str` directly is perfectly acceptable.  When a global variable is declared, it implicitly possesses the longest possible lifetime, ensuring its availability throughout the program's execution.

Demonstrating the declaration and usage of a global variable:

```rust
static NAME: &str = "GLOBAL VALUE";

fn main() {
    println!("{}", NAME);
}

// Output
// GLOBAL VALUE
```

## 3. Constants

Constants are defined using the `const` keyword and, like global variables, are conventionally named using all capital letters.

```rust
const PI: f32 = 3.14;

fn main() {
  println!("{}", PI);
}

// Output
// 3.14
```

Rust provides built-in constants that can be accessed via the `use` keyword. For instance, to use the built-in PI constant, import it from the `std::f32::consts` module.

```rust
use std::f32::consts;

fn main() {
  println!("{}", consts::PI);
}

// Output
// 3.1415927
```

## 4. Values

- **Number Literals:**

> Hexadecimal: `0x` (e.g., `0x46` = 70)
> 

>

> Octal: `0o` (e.g., `0o106` = 70)
> 

>

> Binary: `0b` (e.g., `0b1000110`)
> 
- **Underscores:**

> Underscores can be inserted between digits for improved readability.
> 

>

> Example: `1_000_000`
> 
- **Integers:**

> `i8`, `i16`, `i32`, `i64`: The default integer type is `i32`.
> 
- **Floating-Point Numbers:**

> `f32`, `f64`: The default floating-point type is `f64`.
> 

**Important:** Floating-point variables *must* include a decimal point.

- **Boolean:**

```rust
let t = true;
let f: bool = false; // with explicit type annotation
```

- **Character:**

```rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
```

> Characters are enclosed in single quotes. Declaring a single character infers the `char` type.
> 
- **Tuple:**

```rust
let tup: (i32, f64, i16) = (500, 6.4, 1);
let (x, y, z) = tup;
println!("All Values: {} {} {}", tup.0, tup.1, tup.2);
println!("The value of y is: {}", y);
```

> Tuples allow you to group multiple values of potentially different types together. This is similar to the concept in TypeScript.
To access elements directly, use dot notation with the index (e.g., `tup.0`, `tup.1`, `tup.2`).
> 
- **Array:**

```rust
let a = [1, 2, 3, 4, 5];
let b: [i32; 5] = [1, 2, 3, 4, 5]; // Explicit declaration: array of 5 i32 elements
let c = [3; 5]; // Creates an array of 5 elements, all initialized to 3 (equivalent to [3, 3, 3, 3, 3])
println!("{}", a[0]);
```

## 5. Type Checking

- **Variable Copying:**

While string literals can be assigned directly (similar to JavaScript), numeric and other types require borrowing using the `&` operator.

```rust
fn main() {
  let a = "A";
  let b = a;

  let c = 10;
  let d = &c;
  println!("{} {}", b, d);
}
```

- **String Concatenation:**

Rust does not allow direct string concatenation using the `+` operator with `&str` types.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let _a3 = a1 + a2;
 }

 // Error:
 // `+` can't be used to concatenate two `&str` strings
```

One solution is to convert one of the strings to a `String` type using the `to_string()` method.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let a3 = a1.to_string() + a2;
    println!("{}", a3);
 }
```

Alternatively, you can use the `format!` macro, which functions similarly to `println!`.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let _a3 = format!("{}{}", a1, a2);
 }
```

- **Type Mismatches:**

Rust is strongly typed, meaning that variables of different types cannot be directly assigned to each other, regardless of the underlying memory representation.

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: u32 = 0;
 mutable_num = num; // Error!
}

// Error:
// expected u32, found i32
```

The `as` keyword can be used to perform type conversions between compatible types (e.g., integer to float). However, it cannot be used to convert between unrelated types (e.g., integer to `&str`).

// Similar Type Conversion (i32 -> u32)

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: u32 = 0;

 mutable_num = num as u32;

 println!("{}", mutable_num);
}

// Result
// warning: unused assignment
// help: maybe it is overwritten before being read?
// 10
```

While the code compiles and executes, the compiler issues a warning about the type conversion.

// Dissimilar Type Conversion (i32 -> &str)

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: &str = "";

 mutable_num = num as &str;

 println!("{}", mutable_num);
}

// Result
// Error:
// an `as` expression can only be used to convert between primitive types. Consider using the `From` trait
```

An error occurs when attempting to convert between incompatible types.

- **Custom Type Definitions:**

Rust allows you to define custom types based on existing primitive types using the `type` keyword.

```rust
type Custom = i64;

fn main() {
 let run: Custom = 100;
}
```
