---
title: "강좌07-Ownership"
date: 2019-11-11 17:01:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---



Rust는 Ownership이라는 개념이 존재합니다.<br>
Ownership은 값이 담긴 메모리의 실제 소유자(사용자)가 누구인가를 의미합니다.<br><br>

예를 들어 a라는 변수에 값을 선언하였으나 이를 함수에서 사용하기 위해 전달하였다면 Ownership은 a에서 함수로 옮겨 간 것이며, a는 실제 소유자가 아닌 것입니다.<br><br>

Ownership은 Reference와 Borrow에 영향이 있으므로 중요한 개념입니다.<br>
이를 결정하는 요소인 Copy / Drop에 대해 알아보며 이해해보도록 합시다.<br>


### Shallow Copy and Deep Copy 

- Shallow copy: 같은 주소에 있는 값을 복사하여 한쪽의 변수가 값을 변경할 경우 다른 변수도 영향을 받는 것을 의미합니다.

- deep copy: 같은 값을 새로운 주소에 복사하여 한쪽 변수가 값을 변경해도 다른 변수에 영향을 끼치지 않습니다.

Copy는 문자형 변수는 그대로 변수명을 지정할 경우,<br>
숫자형이나 기타 값을 갖는 경우에는 &로 지정해주어야 가능합니다.<br>

```rust
  let s1 = "hello";
  let s2 = s1;
  
  let s3 = 10;
  let s4 = &s3;
  
  println!("{:p} {:p}", s1, s2);

  //결과
  0x7ff65b76d2e0 0x7ff65b76d2e0
```
  
위의 결과는 같은 주소를 가지지만 변수의 값을 변경하면 새로운 주소를 할당받아 그곳에 새 값을 넣습니다.<br>
즉, 일반적인 복사는 Shallow Copy가 아닌 Deep Copy가 발생함을 확인할 수 있습니다.<br>

  ```rust
  let s1 = "hello";
  let mut s3 = s1;
  s3 = "hallow";
  println!("{:p} {} {:p} {}", s1, s1, s3, s3);
  
  //결과 
  0x7ff6b992d2e0 hello 0x7ff6b992d2e5 hallow
  // s1의 값을 변경해도 결과는 같습니다.
```

String::from 을 사용하면 변수는 기존 문자열과는 첫번째 글자의 주소값이 아닌 달리 값을 가지고 있으므로 이 변수를 그대로 copy할 수 없습니다.<br><br>

이럴 때는 .clone() 명령을 써서 값을 복사해야 합니다. (Deep Copy)<br>
이 때는 &str과는 다르게 처음부터 다른 주소에 copy 됨을 확인할 수 있습니다.

```rust
  let t1 = String::from("hello");
  let t2 = t1.clone();
  println!("{:p} {:p}", &t1, &t2); // 값을 가진 변수이므로 주소를 가져오기 위해 &를 붙여야 합니다.

0xb0a90ff7b8 0xb0a90ff7d0
```


### Ownership과 Function

Javascript에서는 변수의 값을 함수에 넘긴 이후에도 변수를 사용하는데에 문제가 없으나 Rust에서는 Ownership 이 발동하므로 유의하여야 합니다. 


```rust
fn main() {
    let t1 = String::from("hello");
    str_ownership(t1);

    println!("t1 {}", t1);
}

fn str_ownership(s: String) {
    println!("str_ownership {}", s);
}

//에러발생
  |
2 |     let t1 = String::from("hello");
  |         -- move occurs because `t1` has type `std::string::String`, which does not implement the `Copy` trait
3 |     str_ownership(t1);
  |                   -- value moved here
4 | 
5 |     println!("t1 {}", t1);
  |                       ^^ value borrowed here after move
```

위의 예에서 변수 t1의 값 "hello"가 함수로 전달된 시점에서 Ownership이 변수t1에서 함수 str_ownership로 넘어갔으므로(borrowed) 함수 호출 이후 t1 변수를 사용할 수 없습니다.<br><br>


그러나 모든 케이스에서 발생하는 현상은 아닙니다.<br>
integer나 몇몇 형의 경우 함수에 전달할 때 ownership이 넘어가지 않고 copy가 발생하여 함수 호출 이후에도 사용이 가능합니다.

```rust
fn main() {
    let t1 = 10;
    int_ownership(t1);

    println!("t1 {}", t1);
}

fn int_ownership(s: i32) {
    println!("int_ownership {}", s);
}

//결과
int_ownership 10
t1 10
```



### Copy

먼저 Copy의 특징이 무엇인지 알아보고, 어떠한 케이스에서 Copy가 발생하는지 알아보겠습니다. <br><br>

Copy의 특징은 Scope를 벗어나도 Drop이 발생하지 않아 메모리에 상주한다는 것입니다.<br>
즉, Ownership에 영향이 없음을 의미합니다.<br><br>

Copy가 발생하는 조건은 아래와 같습니다.

- All the integer types, such as u32.
- The Boolean type, bool, with values true and false.
- All the floating point types, such as f64.
- The character type, char.
- Tuples, if they only contain types that are also Copy. For example, (i32, i32) is Copy, but (i32, String) is not.


### Drop

위의 케이스에 속하지 않는 String 같은 케이스는 Scope를 벗어나거나 함수에 값을 전달하면 메모리에서 제거 되므로 이후로 사용할 수 없습니다.<br>
이는 Ownership이 이동(Move) 했기 때문입니다.


```rust
fn main() {
    let s = String::from("hello");
    ownership(s);

    println!("s {}", s);
}

fn ownership(s: String) {
    println!("{}", s);
}

//결과
   |
4 |     let s = String::from("hello");
   |         - move occurs because `s` has type `std::string::String`, which does not implement the `Copy` trait
5 |     ownership(s);
   |               - value moved here
6 | 
7 |     println!("s {}", s);
   |                      ^ value borrowed here after move
```

반면에 copy가 가능한 케이스는 ownership이 넘어가지 않고 copy되므로 Drop되지 않아 계속 사용 가능합니다.

```rust
fn main() {
    let s = 10;
    ownership(s);

    println!("s {}", s);
}

fn ownership(s: i32) {
    println!("{}", s);
}

//결과
10
s 10
```






### 예외사항

#### - &str 형의 copy

&str 형은 shallow copy 되므로 실제 값을 전달하지 않고 주소값만 전달하므로 String형과는 달리 Drop 되지 않습니다.

```rust
fn main() {
    let t1 = "hello";
    str_ownership(t1);

    println!("t1 {}", t1);
}

fn str_ownership(s: &str) {
    println!("str_ownership {}", s);
}

//결과
str_ownership hello
t1 hello
```





### Droping Function Returning Value

함수의 리턴값은 함수를 벗어나면 Scope 영역을 벗어나므로 형에 관계 없이 Drop 됩니다.

```rust
fn main() {
    let s = String::from("hello");
    let (s1, len) = ownership(s);
    println!("{} {}", s1, len);
}

fn ownership(s: String) -> (String, usize) {
    let slen = s.len();
    (s, slen)
}
```

