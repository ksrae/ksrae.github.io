---
title: "강좌06-제어문과 반복문"
date: 2019-11-08 18:41:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


일반적으로 모든 언어에는 if와 switch 같은 제어문과<br>
for나 while과 같은 반복문이 있습니다.<br><br>

rust에도 이와 같은 명령어가 있는데 간단히 알아보겠습니다.<br><br>

rust의 제어문과 반복문의 특징은 조건을 () 내에 포함하지 않으며,<br>
for문은 for..in..을 사용한다는 것입니다.<br><br>

기본적인 명령어들이므로 예제를 위주로 정리해보겠습니다.<br>



### if

```rust
  if number < 5 {
  } else if {
  }
  }else {
  }
```



- int형을 bool 형으로 쓸 수 없습니다.

```rust
  let number = 1;
  if number {
  } else {
  }
  // error 
```


아래와 같은 방식으로 써야 합니다.

```rust
  fn main() {
      let number = 3;

      if number != 0 {
          println!("number was something other than zero");
      }
  }
```



- 결과를 변수에 넣을 수 있습니다.

```rust
  let condition = if number {
    5
  } else {
    6
  };
```

        
- 이 때, if와 else if, else에서 리턴하는 값은 반드시 같은 형이어야 합니다.
  
```rust
  let condition = if number < 0 {
    5
  } else if number == 0 {
    false
  }
  } else {
    "Positive"
  };
  // 각각 int, bool, string 형이므로 안된다. 
```
        

### loop: 조건없이 무제한 반복
  
```rust
  loop {
  }
```

        
#### break

 - break에 수식을 넣어 리턴값으로 활용할 수 있습니다.
 
```rust
  let mut counter = 0;
  let result = loop {
  counter += 1;

    if counter == 10 {
        break counter * 2;
    }
  };
```
   
   
### while: 조건이 있는 loop

```rust
  while index < 5 {
    println!("the value is: {}", a[index]);

    index += 1;
  }
  ```

### for

javascript의 for in만 가능합니다. <br>in 뒤에 범위를 지정하면 in 앞에 선언된 변수에 담겨 for문에서 활용할 수 있습니다.

```rust
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
    
fn five() -> i32에서 i32가 리턴형을 의미합니다.<br>



 
  
### Attribute

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


#### Testing
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