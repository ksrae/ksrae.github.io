---
title: "강좌08-Reference and Borrowing"
date: 2019-11-19 17:16:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


이제 Reference와 Borrowing에 대해 알아보겠습니다.

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


위의 코드의 경우 s.len()을 수행할 수 없는데 이는 Ownership이 넘어갔기 때문임을 알고 있습니다.


#### Reference

javascript에서는 함수에 인자를 전달할 때 by Reference와 by Value로 전달할 수 있습니다.<br>
이 두 가지를 테스트 해보겠습니다.


- by reference

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

by reference의 경우 주소만 넘기므로 ownership이 넘어가지 않습니다. <br>따라서 함수 이후에도 해당 변수를 사용할 수 있습니다.



- by value

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


by value는 값을 넘기므로 ownership을 넘기는 것이며 따라서 함수 이후에 해당 변수는 Drop 됩니다.


#### Mutable Reference 

위의 두 케이스 모두 인자가 immutable이므로 함수 내에서 값을 수정할 수 없는데 이를 해결하려면 두 가지 방법이 있습니다.

- 다른 변수에 할당하여 사용합니다.<br>
- &mut 형태로 전달합니다.


두번째 방법에 대해 알아보겠습니다. <br>
&mut 형태로 전달하려면 변수는 mutable 형태여야 합니다.<br><br>

예를 통해 쉽게 알아보겠습니다.

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


&는 reference 할 수 있도록 해주는 특수 문자 입니다.<br>
반대의 경우는 * 가 있는데 c와 같은 방식 입니다.


### Borrow

위의 예시에서 &mut은 주소를 참조하는 것이 아니라 borrow 하는 것인데<br>
이는 &를 사용한 reference와는 또 다른 방식입니다.<br><br>


- Ownership: 변수가 더이상 메모리에 존재하지 않는 상태.<br>
- Borrow: mutable reference. 변수는 존재하나 Ownership이 다른 변수/함수에 넘어간 상태. Scope가 넘어간 상태에서 가능. &mut<br>
- Reference: immutable reference. Ownership에 관계 없이 변수를 참조한 상태.




다중 immutable reference는 가능

```rust
let mut s = String::from("hello");

let r1 = &s; 
let r2 = &s; 
```

immutable reference가 완료되지 않으면 borrow는 불가능

```rust
let mut s = String::from("hello");

let r1 = &s; 
let r2 = &mut s; 
//에러 발생
```

reference를 사용한 뒤에 borrow는 가능
```rust
let mut s = String::from("hello");

let r1 = &s; 
println!("{}", r1);

let r2 = &mut s; 
// 가능
```

함수에 넘긴 뒤에도 borrow 가능

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

함수에 넘겼지만 다시 reference로 리턴하면 borrow 할 수 없다.

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

// 에러
```

value로 리턴하면 borrow할 수 있다.

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

// 성공
```

반대로 borrow중일 때 reference도 안된다.

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
//에러
  |
3 |     let r1 = print(&mut s);
  |                    ------ mutable borrow occurs here
4 | 
5 |     let r3 = &s;
  |              ^^ immutable borrow occurs here
6 |     println!("s {} {}", r1, r3);
  |                         -- mutable borrow later used here
```


역시 borrow 중일 때 borrow도 안됨

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s; 

    println!("{} {}", r1, r2);
}

// 결과
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 | 
7 |     println!("{} {}", r1, r2);
  |                       -- first borrow later used here
```

결론: 
- reference -> reference O
- reference -> borrow X
- borrow -> reference X
- borrow -> borrow X
- reference / borrow 종료 후 -> reference / borrow O


### Dangling References

lifetime이 종료되면 reference / borrow가 불가능합니다.<br><br>

함수 내부 변수를 외부로 전달할 때 reference로 전달할 수 없습니다. <br>함수가 종료되면 원본 변수가 Drop되어 reference할 수 없기 때문입니다.


```rust
fn main() {
    let r1 = dangling();

    println!("{}", r1);
}

fn dangling() -> (&String) {
    let s = String::from("hello");
    &s
}
// 결과
   |
7 | fn dangling() -> (&String) {
   |                   ^ help: consider giving it a 'static lifetime: `&'static`
   |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed
```

이럴 때는 값을 직접 리턴해야 합니다.

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


## Reference 요약 정리
- At any given time, you can have either one mutable reference or any number of immutable references.<br>
- References must always be valid.
















