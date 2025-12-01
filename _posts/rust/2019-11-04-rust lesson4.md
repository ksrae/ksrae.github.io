---
title: "Lesson 04-Output"
date: 2019-11-04 14:46:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C04-%EC%B6%9C%EB%A0%A5)
<br/>


### Printing to the Console with `println!`

The `println!` macro is a fundamental tool in Rust for displaying output to the console. This section explores its capabilities, covering various output formatting options.

- **Printing Numbers and Text**

As with most programming languages, numbers are printed directly, while text requires enclosure in double quotes.

```rust
fn main() {
    println!("{}", 100);
    println!("{}", "This is Text");
}
```

- **Variable Interpolation**

To print the value of a variable, designate its position within the output string using `{}` placeholders. For printing multiple variables, while direct use of `{}` is possible, leveraging indexed placeholders like `{n}` (where *n* is a number) allows for variable reuse within the format string.

```rust
fn main() {
    let pi: f32 = 3.14;
    println!("{}", pi);

    let txt: &str = "Test";

    println!("this is {0} {1} {1}", pi, txt);
}

// Result:
// 3.14 Test Test
```

- **Inline Variable Declaration**

`println!` supports direct variable declaration within its scope. The syntax `{variable_name=value}` declares and initializes a variable directly in the format string.

```rust
fn main() {
    println!("this is {pi} {txt}", pi = 3.14, txt = "Text");
}
```

It's crucial to note that these inline variables are only valid within the scope of the `println!` macro, making them single-use only. Attempting to access them outside this scope will result in an error.

```rust
fn main() {
    println!("this is {pi} {txt}", pi = 3.14, txt = "Text");
    println!("this is {}", pi); // Error: `pi` not found in this scope.
}

/*
  |
4 |   println!("this is {}", pi);
  |                             ^^ not found in this scope

*/
```

Due to the limited scope and potential debugging difficulties, this approach is generally discouraged in production code.

- **Formatted Output**

The `println!` macro allows for fine-grained control over how variables are formatted for output. By appending a colon (`:`) followed by a format specifier to the variable placeholder, you can specify the desired output representation (e.g., hexadecimal, binary, exponential).

| Specifier | Description                         |
| :-------- | :---------------------------------- |
| `:o`      | Octal representation                |
| `:x`      | Hexadecimal representation (lowercase) |
| `:X`      | Hexadecimal representation (uppercase) |
| `:p`      | Pointer address                     |
| `:b`      | Binary representation               |
| `:e`      | Exponential notation (lowercase)      |
| `:E`      | Exponential notation (uppercase)      |
| `:?`      | Debug representation                |

The `format!` macro shares the same formatting syntax as `println!`, but instead of printing to the console, it returns the formatted string as a value. This is useful for constructing strings for later use.