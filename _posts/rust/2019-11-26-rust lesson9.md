---
title: "강좌09-Slice Type"
date: 2019-11-26 17:16:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


### 정의

데이터의 연속된 값을 참조하는 데이터 타입을 Slice Type 이라고 합니다.<br>
만일 문자열 있다고 할 때, Slice 타입은 이 문자열의 부분을 담을 수 있습니다.<br>
javascript에서도 이러한 부분 문자열을 자를 수 있는 함수가 존재합니다.

```javascript
let a = "hello world";
let slice = a.substring(0,5);
```

rust 에서는 이러한 값을 Slice Type을 활용하여 잘라낼 수 있습니다.


```rust
let a = String::from("hello world");
let slice = &a[0..5];
```
위의 예에서는 변수 slice는 "hello" 값을 가지며, &str 타입이 됩니다.<br>


### 활용

Slice Type은 아래와 같이 사용하며, 다양한 방법으로 활용할 수 있습니다.

```
1. 첫번째 값과 끝값은 생략 가능합니다.
[0..5] -> [..5], [5, (문자의 총 길이)] -> [5..]

2. 전체 -> [..] 로 표현할 수도 있으며 배열에서도 가능합니다.
```


다음은 문자를 받아 앞 5글자만 리턴하는 함수 입니다.<br>
이 예제에서 주목할 부분은,<br>
 함수와 <strong>&변수[..]</strong> 로 String 형을 &str로 전달하는 방법 입니다.

```rust
fn main() {
    let s1: &str = "hello world";
    let word = first_word(s1);
    
    let s2 = String::from("hello world");
    let word2 = first_word(&s2[..])
}

fn first_word(s: &str) -> &str {
    &s[0..5]
}
```

아래의 예제는 위의 함수를 확장하여 문자열을 받으면 첫번째 문자부터 스페이스를 만날 때까지를 리턴하는 함수입니다.


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


