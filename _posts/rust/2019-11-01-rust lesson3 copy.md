---
title: "3. 변수"
date: 2019-11-01 11:00:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


### 1. immutable과 mutable 선언

```rust
  let x: i32 = 1;
  // 형선언은 생략 가능. 생략하면 컴파일러가 판단하여 적용.
```

    
기본 변수 선언 방식인 let으로 x를 정의하면 x는 immutable (변경할 수 없는 값) 이 되므로, <br>선언한 이후에 x = 2; 와 같이 값을 변경할 수 없습니다.<br><br>

다시말해 기본 선언이 readonly가 된다고 보면 쉬울 것 같습니다.

```rust
  let x = 1; 
  x = 2;
  // error
```


수정 가능한 변수로 만들기 위해 mut로 선언해야 합니다. <br>let 다음에 mut를 붙이면 mutable 상태로 만들 수 있으며 말 그대로 변수가 됩니다.

```rust
  let mut x = 1; 
  x = 2;
  // x = 2
```


#### 예외
 
예외 적으로 mut 없이도 값을 변경할 수 있는 케이스가 있는데 <br>같은 형을 가진 상태로 재선언 하는 것 입니다. (같은 주소에 값이 재설정 됩니다.)

```rust
  let x = 1;
  let x = x+1; 
  let y = y+1;
  let y = x+2;
```


같은 변수명이라도 다른 형을 가지면 mut로 선언해도 변경이 불가능합니다.<br>
즉, mut는 단순히 readonly를 해제하는 것이지 변수의 형까지 변경하지는 않습니다.

```rust
  // 가능
  let spaces = "   ";
  let spaces = spaces.len();

  //불가능
  let mut spaces = "   ";
  spaces = spaces.len();
```


    
### 2. Scope

javascript와 유사하게 변수의 범위가 제한됩니다.<br>
immutable 변수라도 scope 위치가 다르면 다른 변수로 인식하여 선언할 수 있습니다. 

```rust
fn main() {
  let outer = 1;
  {
    let outer = 2;
    let inner = 3;
    println!("{} {}", outer, inner);
  }
  println!("{}", outer);
}

//결과
2 3
1
```

이 때는 위의 재선언과 달리 다른 주소에 저장된 완전히 다른 변수로 인식합니다. <br>
(뒤에 나올 Ownership에 따라 Scope을 벗어나면 Drop 되므로 메모리에서 제거 됩니다.)


4. 값과 주소

만일 변수를 c의 포인터와 같이 다른 변수와 같은 주소를 공유하는 용도로 활용하려면 값에 &를 붙여야 합니다.

```rust
fn main() {
    let t1 = 10;
    let t2 = &t1;
    println!("{:p} {:p}", &t1, t2);
 }
 
 //결과
 0x77114ff5f4 0x77114ff5f4
 ```
 
  
 만일 일반 변수 복사하듯 선언하면 copy가 발생하며 같은 주소를 공유하지 않습니다.
 
 ```rust
fn main() {
    let t1 = 10;
    let t2 = t1;
    println!("{:p} {:p}", &t1, &t2);
    
// 결과
0x5e368ff820 0x5e368ff824
 ```
 
 c처럼 포인터 *를 사용하지 않아도 출력 옵션에 따라 값과 주소를 모두 표현할 수 있습니다.
 
 