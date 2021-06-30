---
title: "강좌05-함수"
date: 2019-11-06 18:22:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


함수의 기본 사용법은 javascript와 유사하나 rust에서는 function 대신 fn을 사용합니다.<br>

```rust
  fn main() {
  }
```

    
### 1. 파라미터

```rust
  fn main() {
      another_function(5);
  }

  fn another_function(x: i32) {
      println!("The value of x is: {}", x);
  }
```

    
반드시 파라미터에 형을 정의하여야 합니다.<br> 그렇지 않으면 에러가 발생하므로 javascript 개발자는 유의하여야 합니다.<br>


### 2. 리턴

rust에서 리턴은 javascript와 달리 return을 생략하고 세미콜론(;)도 빼야 합니다. <br>

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
    
fn five() -> i32에서 i32가 리턴형을 의미합니다.<br>



 
  
### 3. Attribute

- 조건부 실행

#[cfg(조건)] 을 사용하면 조건이 허용될 때만 함수가 실행됩니다.<br>
아래 예시는 windows 일 때만 실행되며, 아닐 경우 에러가 발생하는 코드입니다.


```rust
fn main() {
 on_windows();
}

#[cfg(target_os = "windows")]
fn on_windows() {
 println!("This OS is windows.")
}

// 결과 - os가 windows 일 때,
This OS is windows.

// 결과 - os가 linux 일 때,
   |
2 |  on_windows();
   |  ^^^^^^^^^^ not found in this scope
```


- Testing
#[test] 로 사용하며 함수가 수행 가능한지 테스트합니다.<br>
rustc나 cargo로 다음과 같이 테스트할 수 있습니다.

```bash
rustc --test [파일명]
```

또는

```bash
cargo test
```


#[] 문법에 대해서는 나중에 더 자세히 다루도록 하겠습니다.