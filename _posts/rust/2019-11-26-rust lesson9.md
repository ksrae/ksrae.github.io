---
title: "Lesson 09-Slice Type"
date: 2019-11-26 17:16:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EA%B0%95%EC%A2%8C09-Slice-Type)
<br/>

## Definition

In Rust, a Slice Type refers to a data type that references a contiguous sequence of elements within a collection. When dealing with strings, for example, a Slice Type can hold a portion of that string, providing a view into the underlying data without copying it. This concept is similar to extracting substrings in JavaScript.

```jsx
let a = "hello world";
let slice = a.substring(0,5);
```

Rust's Slice Types offer a robust mechanism for achieving the same outcome, leveraging references for efficiency.

```rust
let a = String::from("hello world");
let slice = &a[0..5];
```

In the example above, the `slice` variable holds the value `"hello"` and is of type `&str`, representing a string slice.

## Usage

Slice Types are versatile and can be used in several ways, as illustrated below:

```
1.  The start and end indices can be omitted.
    `[0..5]` can be shortened to `[..5]`, and `[5, (total length of the string)]` can be written as `[5..]`.

2.  The entire range can be represented as `[..]`, which is applicable to both strings and arrays.
```

The following code defines a function that takes a string and returns the first five characters as a slice. Key points to note in this example include: (1) the function signature `fn first_word(s: &str) -> &str` indicates that it accepts a string slice and returns a string slice and (2) the method of passing a `String` type as a `&str` using **`&variable[..]`**.

```rust
fn main() {
    let s1: &str = "hello world";
    let word = first_word(s1);

    let s2 = String::from("hello world");
    let word2 = first_word(&s2[..]);
}

fn first_word(s: &str) -> &str {
    &s[0..5]
}
```

Building upon the previous example, the subsequent code showcases a function that, given a string, returns the portion from the beginning up to the first space encountered. This function elucidates how to iterate through the bytes of a string and identify a specific delimiter.

```rust
fn main() {
    let s = String::from("hello world");
    let word = first_word(s);
}

fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```