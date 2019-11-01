---
title: "2. 주석과 변수 타입"
date: 2019-10-31 18:38:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---


### 1. 주석

주석달기는 javascript의 주석과 같습니다. <br>

```rust
// 한줄 주석
/*
여러줄 주석
*/
/// 문서화에 사용할 주석
```

Rust에서는 여러줄이라도 한줄 주석으로 만들 것을 추천하며,<br>
문서화에 필요한 주석은 /// 을 사용 합니다.<br>


## 2. 전역 변수

전역 변수는 static을 사용하며 변수명은 대문자로 작성해야 합니다.

```rust
static MAX: i32 = 100;
```

문자열은 &str 형을 사용하는데

```rust
static NAME: &str = "GLOBAL VALUE";
```

<del>문자형 전역 변수의 lifetime 유지를 위해 &'static str 형을 사용하여야 합니다.</del><br>
<del>이는 lifetime 때문인데 'static 형은 가장 긴 lifetime을 가지므로 적합하다 볼 수 있습니다.</del><br><br>

&str 형을 그대로 사용하여도 됩니다. 전역 변수로 선언하면 가장 긴 lifetime을 갖게 됩니다.<br><br>

위의 변수를 선언하여 이를 출력하면 정상적으로 출력됨을 볼 수 있습니다.

```rust
static NAME: &str = "GLOBAL VALUE";

fn main() {
    println!("{}", NAME);
}

//결과
GLOBAL VALUE
```



### 3. 상수

상수는 const를 사용하며 변수명은 대문자로 작성합니다.

```rust
const PI: i32 = 3.14;

fn main() {
  println!("{}", PI);
}

//결과
3.14
```

Rust에 내장된 상수를 use를 통해 사용할 수도 있습니다.<br>
예를 들어 내장된 PI 상수를 사용하려면 std::f32::consts를 use 해야합니다.<br>

```rust
use std::f32::consts;

fn main() {
  println!("{}", consts::PI);
}

//결과
3.1415927
```


### 4. 값

- 진수 
> 16진수: 0x  (0x46 = 70) <br>
> 8진수: 0o (0o106 = 70) <br>
> 2진수: 0b (0b1000110) <br>

- 언더스코어 (_)
> 가독성을 위해 숫자 사이에 _를 넣을 수 있습니다. <br>
> 예: 1_000_000 <br>
 
- 정수
> i8, i16, i32, i64 : default i32 <br>

- 실수
> f8, f16, f32, f64: default f64 <br>

** 실수 변수는 반드시 소수점을 포함해야 합니다.

- boolean

```rust
let t = true;
let f: bool = false; // with explicit type annotation
```


- char

```rust
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
```

> 싱글 따옴표를 사용하며 문자 하나만 선언하면 char 형으로 인식합니다.


- Tuple

```rust
    let tup: (i32,i32, i16) = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("All Values: {} {} {}", tup.0, tup.1, tup.2);
    println!("The value of y is: {}", y);
```

> 여러 값을 묶어서 선언할 수 있습니다. 타입스크립트의 방식과 유사합니다.<br>
> tup에서 직접 값을 가져오려면 tup.0, tup.1, tup.2 로 각각 가져올 수 있습니다.


- Array

```rust
    let a = [1, 2, 3, 4, 5];
    let b: [i32; 5] = [1, 2, 3, 4, 5]; // i32형 5개 선언 정의
    let c = [3; 5]; // 3값을 5개. 즉, [3,3,3,3,3]과 동일
    println!("{}", a[0]);
```


### 5. 타입 체크


- 변수의 복사

문자열의 경우 javascript와 같이 변수명을 직접 적용해도 되나,
숫자형이나 기타 다른 형의 경우 &로 가져와야 합니다.

```rust
fn main() {
  let a = "A";
  let b = a;

  let c = 10;
  let d = &c;
  println!("{} {}", b, d);
}
```


- string 합치기

Rust에서는 string을 합칠 때 + 를 사용할 수 없습니다.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let _a3 = a1 + a2;
 }
 
 //결과
  |
4 | let a3 = a1 + a2;
  |               ^^^^^^^^^^^^^^^^^ `+` can't be used to concatenate two `&str` strings
```

한쪽 변수를 to_string() 형태로 변경하면 가능합니다.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let a3 = a1.to_string() + a2;
    println!("{}", a3);
 }
```


또는 format!을 활용할 수도 있습니다. 사용법은 println!과 같습니다.

```rust
fn main() {
    let a1 = "Kim";
    let a2 = "Lee";
    let _a3 = format!("{}{}", a1, a2);
 }
```


- 다른 형은 서로 복제할 수 없습니다.

javascript와는 달리 형을 강하게 따지므로 형이 다르면 값을 복사할 수 없습니다. <br>이는 형의 범위나 메모리 할당의 크기와 관계 없습니다.

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: u32 = 0;
 mutable_num = num; // error !
}

//결과
  |
4 |  mutable_num = num; // error !
  |                ^^^ expected u32, found i32
```

as를 활용하면 유사한 형 (int, float 등)은 서로 인식이 가능합니다.<br>
그러나, 유사하지 않은 형 (int, &str)은 여전히 인식 불가합니다.<br><br>

// 유사한 형 변환 (i32  -> u32)

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: u32 = 0;

 mutable_num = num as u32;

 println!("{}", mutable_num);
}

// 결과
  |
3 |  let mut mutable_num: u32 = 0;
  |          ^^^^^^^^^^^
  |
  = note: #[warn(unused_assignments)] on by default
  = help: maybe it is overwritten before being read?
  
10
```

유사항 형을 as로 강제로 복사했더니 결과값은 나오지만 형 변환에 따른 경고 메시지가 나옵니다.<br><br>

// 유사하지 않은 형 변환 (i32 -> &str)

```rust
fn main() {
 let num: i32 = 10;
 let mut mutable_num: &str = "";

 mutable_num = num as &str;

 println!("{}", mutable_num);
}

//결과
  |
5 |  mutable_num = num as &str;
  |                ^^^^^^^^^^^
  |
  = note: an `as` expression can only be used to convert between primitive types. Consider using the `From` trait
```

유사하지 않은 경우 에러가 발생하고 진행되지 않습니다.


- Custom 형 정의

기본형을 활용한 Custom 형을 정의하여 이를 활용할 수 있습니다.

```rust
type Custom = i64;

fn main() {
 let run: Custom = 100;
}
```
