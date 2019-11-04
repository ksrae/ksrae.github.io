---
title: "강좌04-출력"
date: 2019-11-04 14:46:00 +0900
comments: true
categories: rust
tags: [lesson-rust]
---




### Println!

화면에 출력하는 함수 입니다.

- 숫자/텍스트 출력

일반적인 출력 값과 같이 숫자는 그대로, 텍스트는 따옴표로 묶어 출력합니다.

```rust
fn main() {
  println!(100);
  println!("This is Text");
}
```

- 변수 출력

변수를 출력할 위치를 {} 로 지정합니다.<br><br>
다중 변수를 출력하려면 {}를 직접 써도 되나<br> {n} (n은 숫자)로 활용하면 변수를 재활용하여 사용할 수 있습니다.<br>

```rust
fn main() {
  let pi: f32 = 3.14;
  println!("{}", pi);
  
  let txt: &str = "Test";
  
  println!("this is {0} {1} {1}", pi, txt);
}

//결과
3.14 Test Test
```

- 직접 변수 선언

println 내부에 직접 변수를 선언하고 변수명을 {} 안에 입력하여 출력할 수 있습니다.

```rust
fn main() {  
  println!("this is {pi} {txt}", pi=3.14, txt="Text");
}
```

1회성이므로 이후로는 재사용할 수 없으니 유의해야 합니다.

```rust
fn main() {  
  println!("this is {pi} {txt}", pi=3.14, txt="Text");
  println!("this is {}", pi); // 1회성이므로 여기에서 에러가 발생한다.
}

/*
  |
4 |   println!("this is {}", pi);
  |                             ^^ not found in this scope
  
*/
```

실제 코딩에서는 디버깅이 어려우므로 비추합니다.


- 포멧 형식 출력

변수를 진수, 바이너리 등으로 출력하려면 : 뒤에 정의된 값을 입력합니다.

| 표기 | 값 | 
|:--|:--|
| :o	| 8진수 |
| :x	| 16진수 (소문자로 표기) |
| :X	| 16진수 (대문자로 표기) |
| :p	| 포인터 |
| :b	| 2진수 |
| :e	| 지수  (소문자로 표기) |
| :E	| 지수 (대문자로 표기) |
| :?	| 디버깅 목적 |

format! 명령도 println! 과 같은 형식을 취하나 format!은 출력이 아닌 값을 리턴하는 용도로 사용합니다.

